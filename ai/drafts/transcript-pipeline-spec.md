# Spec: Audio Transcription & Cleanup Pipeline

Personal AI Ops repo. Takes Obsidian-recorded audio through local transcription and a
glossary-aware LLM cleanup pass. Durable knowledge extraction is **out of scope** — it runs
downstream against the cleaned output.

**Capture is already configured:** Advanced Audio Recorder plugin, multi-track mode, two tracks,
**16 kHz mono FLAC**, one file per track. Track inputs are the RØDECaster Chat output (mic,
mix-minus) and a RØDECaster sub-mix carrying computer/far-end audio only.

---

## 1. Core design principle

**Deterministic work goes in a shell script. The LLM does exactly one thing: the cleanup pass.**

Do not let the agent drive `whisper-cli`. An agent improvising flags is slow, nondeterministic,
costs tokens, and produces transcripts you can't reproduce. The agent's entire job is
text-in/text-out.

| Stage | Executor | Deterministic? |
|---|---|---|
| 1. Discovery & session grouping | script | yes |
| 2. Transcribe (VAD + `whisper-cli`) | script | yes |
| 3. Merge tracks | script | yes |
| 4. Cleanup | Copilot CLI custom agent | no |

Stages 1–3 batch freely — local, free, repeatable. Stage 4 is gated per session.

**No `ffmpeg`.** Capture already produces 16 kHz mono FLAC, which `whisper-cli` reads natively
(supported inputs: flac, mp3, ogg, wav). There is no format normalization step and no channel
splitting — the plugin writes separate files per track, so the sources arrive already isolated.

---

## 2. Parameters

```sh
VAULT_ROOT="…"                  # Obsidian vault
AUDIO_DIR="$VAULT_ROOT/…"       # Advanced Audio Recorder output dir
OPS_REPO="…"                    # personal AI Ops repo
GLOSSARY="$OPS_REPO/…/glossary.md"
HOT_TERMS="$OPS_REPO/…/hot-terms.txt"
WHISPER_MODEL="…/ggml-large-v3.bin"
VAD_MODEL="…/ggml-silero-v6.2.0.bin"    # required, see §6
WHISPER_THREADS=8

SELF_TRACK="…"                  # track suffix carrying your mic — VERIFY, see §5
RETAIN_AUDIO_DAYS=30            # see §9
```

---

## 3. Repo layout

```
<OPS_REPO>/
├── .github/agents/
│   └── transcript-cleanup.agent.md     # the custom agent profile
├── scripts/
│   ├── transcribe.sh                   # stages 1–3
│   └── cleanup.sh                      # stage 4 wrapper
└── <glossary location>                 # already exists
```

Custom agent profiles live in `.github/agents/` and must end in `.agent.md`. Filename minus
extension becomes the agent name unless `name:` is set in frontmatter. Prompt body caps at
30,000 characters.

---

## 4. File conventions

A **session** is one recording. It has one or two source FLAC files depending on mode.

| File | Role | Mutable? |
|---|---|---|
| `<session>.<track>.flac` | source audio, one per track | never |
| `<session>.<track>.raw.json` | per-track whisper output | regenerable |
| `<session>.raw.txt` | merged, speaker-labeled transcript | **never** — audit trail |
| `<session>.clean.md` | cleaned transcript | regenerable |
| `<session>.meta.json` | provenance | regenerable |

`meta.json` records: whisper model, track count, track→speaker mapping, run timestamps, glossary
SHA. The glossary hash is what lets you selectively re-run cleanup after a glossary update
instead of reprocessing everything.

**Idempotency:** `.raw.txt` exists → skip transcription. `.clean.md` exists → skip cleanup.
`--force` overrides. Sidecar existence is the "already transcribed?" check — nothing cleverer.

Per-track `.raw.json` files are intermediates. Keep them until the merge succeeds, then delete
or leave them; they're cheap and useful for debugging a bad merge.

---

## 5. Session grouping and track identity

This is the one genuinely new piece of logic, and the one thing to verify empirically before
writing the script.

**Grouping.** The script must know which files belong to the same session. Confirm what the
plugin actually emits — a shared basename with a track suffix, a per-session subfolder, or
timestamped names. Encode that pattern in one function, not scattered through the script.

**Track identity.** Which file is your mic must be determined by a **stable rule**, not by
guessing. Verify once at setup and pin it as the `SELF_TRACK` constant:

1. Record 20 seconds — say "track one, my microphone," then play audio from the computer
2. Listen to each output file
3. Record which suffix corresponds to your mic

Getting this backwards mislabels every transcript silently, and the cleanup pass will not catch
it — the output reads perfectly fine with the speakers swapped. Re-verify after any plugin
update or track-order change.

**Mode.** Derived from file count, not content:

| Files in session | Mode | Behavior |
|---|---|---|
| 1 | solo | one pass, `-nt` (no timestamps), no speaker labels |
| 2 | conversation | two passes with timestamps, merge, deterministic labels |

No `--kind` flag and no directory convention needed — the plugin's track setting already
encodes the distinction. Use single-track mode for voice notes, two-track for calls.

---

## 6. Transcription

**Conversation (two files):**

```sh
whisper-cli \
  -m "$WHISPER_MODEL" \
  -f "$AUDIO_DIR/<session>.<track>.flac" \
  -of "$AUDIO_DIR/<session>.<track>.raw" \
  -l en -t "$WHISPER_THREADS" \
  --vad --vad-model "$VAD_MODEL" \
  --vad-speech-pad-ms 150 \
  --max-context 0 \
  --entropy-thold 2.6 \
  --prompt "$(cat "$HOT_TERMS")" \
  -oj
```

Run once per track. **Solo (one file):** same, but `-otxt -nt` — no timestamps needed, output
goes straight to `<session>.raw.txt`.

### VAD is required, not optional

Per-track capture means each file is silent for roughly half its duration — your mic track while
the other person talks, and the reverse. Long silence is the primary trigger for Whisper
repetition loops, so this architecture makes VAD mandatory rather than a nicety.

Silero VAD marks speech segments first; only those are extracted and passed to Whisper. The
decoder never sees the silence, so it can't get stuck on it. It also roughly halves transcription
time per track.

One-time setup:

```sh
./models/download-vad-model.sh silero-v6.2.0
```

Tunables (defaults in parentheses): `--vad-threshold` (0.5), `--vad-min-speech-duration-ms`
(250), `--vad-min-silence-duration-ms` (100), `--vad-speech-pad-ms` (30),
`--vad-max-speech-duration-s`.

Padding is raised to 150 ms above because the 30 ms default clips word onsets, and a truncated
first syllable costs more accuracy than the extra audio costs time.

**Verify once, before trusting the merge:** confirm that segment timestamps map back to the
*original* audio timeline and not to the concatenated speech-only timeline. The two-track merge
sorts by absolute start time and is completely wrong if VAD returns compacted timestamps. Test
with a recording that has a long, obvious silence and check that a known event lands where it
should.

### Repetition mitigation

`--max-context 0` stops previously decoded text being passed into subsequent chunks. Whisper's
repetition loops are self-reinforcing: a wrong phrase becomes context for the next window and
gets amplified. Disabling the carry-over costs some cross-window continuity, which is an
acceptable trade here because the cleanup pass repairs continuity downstream.

`--entropy-thold 2.6` (default 2.40) makes temperature fallback trigger more readily on
repetitive output. Temperature fallback is on by default in current builds and is the main
built-in defense.

If loops persist after both, try large-v2 — v3 has a documented tendency to repeat more.

**Optional preflight.** If you want a guard against a plugin setting silently changing, assert
sample rate and channel count before transcribing. This needs `ffprobe`, so skip it if you'd
rather have no ffmpeg dependency at all — `whisper-cli` will fail loudly on a malformed input
rather than succeed incorrectly.

**On `--prompt`:** it biases decoding toward supplied vocabulary, capped at `n_text_ctx/2`
(≈224 tokens) — room for perhaps 20–30 terms, not your glossary. Its influence also decays
across a long recording, since the prompt gets displaced by the previous segment's decoded text
in later windows. Published work has found Whisper's prompt-following unreliable and not
positively correlated with accuracy.

Seed `hot-terms.txt` with your highest-frequency proper nouns — team names, product names,
recurring system names. Treat any gain as a bonus. **Acronym correction is the cleanup pass's
job, not whisper's.**

---

## 7. Merge (conversation only)

1. Parse both `.raw.json` files into `(start, end, text, speaker)` records
2. Speaker is assigned by **source file**, not inference: `$SELF_TRACK` → `Curtis`, other → `Other`
3. Sort by `start`
4. Collapse consecutive same-speaker segments into paragraphs
5. Emit `**Speaker:** text` lines to `<session>.raw.txt`

Leave overlapping segments in timestamp order — don't try to interleave mid-utterance.

**Sync note:** the two tracks come from independent device streams, so they may start a few
milliseconds apart. Irrelevant at segment granularity. Don't build anything that assumes
sample-accurate alignment.

---

## 8. Cleanup agent

### 8.1 Agent profile

`.github/agents/transcript-cleanup.agent.md`:

```markdown
---
name: transcript-cleanup
description: Corrects ASR errors and normalizes terminology in meeting and voice-note transcripts. Correction only — never rewrites.
tools: ["read", "write"]
---
```

Deliberately **no shell tool**. The agent has no legitimate reason to run commands, and
withholding it removes a whole class of improvisation.

### 8.2 Agent instructions

**Task:** Read the raw transcript and the glossary. Produce a corrected transcript.

**Permitted:**
- Fix ASR mishearings
- Normalize acronyms and proper nouns to their canonical glossary form
- Remove filler words, false starts, and stutter repetitions
- Apply self-corrections the speaker made ("no wait, I mean…")
- Restore sentence boundaries and paragraph breaks

**Forbidden — state these explicitly and emphatically:**
- Do not paraphrase, reword, condense, summarize, or reorder
- Do not add content, transitions, or clarifications that were not spoken
- Do not change speaker labels or reassign lines between speakers
- Do not invent speaker names — labels come from the pipeline, never from inference
- Do not "improve" phrasing, tone, or professionalism

**Uncertainty rule:** If a term is unclear and no glossary entry plausibly matches, **leave it
verbatim and flag it inline** as `⟨?original text⟩`. Never guess. A confidently wrong
"correction" reads perfectly fine and you will never catch it — this rule is the single most
important line in the profile.

**Output:** the cleaned transcript, followed by:

```markdown
---
## Corrections applied
- "<heard>" → "<corrected>" (glossary: <term>)
- ⟨?unresolved⟩ at ~<timestamp>
```

The corrections log makes the pass auditable. Scan it, and if the agent is "fixing" things you
actually said, tighten the profile.

### 8.3 Invocation

```sh
copilot -sp "Clean the transcript at <RAW>. Glossary: <GLOSSARY>. Write result to <CLEAN>." \
  --agent transcript-cleanup \
  --no-ask-user \
  --allow-tool='write'
```

`-s` strips stats so you capture only the response; `--no-ask-user` prevents blocking on
clarification in a loop.

**Chunking:** single pass under ~90 minutes of audio. Beyond that, split on speaker turns or
paragraph boundaries with a few segments of overlap — never on character count, which produces
mid-sentence seams the model then tries to repair by inventing text.

---

## 9. Retention

16 kHz mono FLAC is roughly 55–65 MB per hour per track, so a two-track hour is ~120 MB. That
still accumulates in a synced vault.

Since the audio exists only as a re-transcription source, add a retention rule rather than
optimizing the encoder further: **delete source FLAC once `.clean.md` exists and is older than
`RETAIN_AUDIO_DAYS`.** Run it as a separate `prune.sh` — never as a side effect of
`transcribe.sh`, so a bug in transcription can't destroy the only copy of the audio.

Keep `.raw.txt` and `.clean.md` indefinitely; they're text.

---

## 10. Invocation surface

```sh
./scripts/transcribe.sh                    # batch: all untranscribed sessions
./scripts/transcribe.sh --session <name>   # single
./scripts/cleanup.sh --session <name>      # stage 4, one session
./scripts/cleanup.sh --all                 # loop over raw-without-clean
./scripts/cleanup.sh --stale               # glossary hash changed since last run
./scripts/prune.sh --dry-run               # retention, list only
```

Trigger manually — from the terminal or an Obsidian Shell Commands hotkey. No watcher, no
scheduler; your processing is interactive by design.

---

## 11. Validation

Before writing `.clean.md`:

- Cleaned word count within roughly 70–110% of raw. Large shrinkage means it summarized instead
  of corrected — **fail loudly and keep the raw file**, don't write the output
- Speaker label set identical between raw and clean
- Every `⟨?…⟩` marker preserved

Before merging:

- Both tracks present for a two-file session. A session with one track where two were expected
  should **fail, not silently transcribe as solo** — that's how you'd lose half a meeting
- Track durations within a few seconds of each other

Per track, after transcription:

- **Repetition detector:** flag the session if the same segment text repeats more than ~3 times
  consecutively, or if any single phrase accounts for an outsized share of total segments. This
  is the loop failure mode; it produces plausible-looking output that runs to the end of the file
- Watch for known trained-in hallucinations ("Thanks for watching", subtitle credit lines) — same
  root cause, and a useful signature to match on
- Flagged sessions should not proceed to cleanup. Re-run with adjusted flags instead

## 12. Acceptance criteria

- Re-running either script is a no-op when sidecars exist
- `.raw.txt` is never modified after creation
- Deleting `.clean.md` and re-running reproduces cleanup; deleting `.raw.txt` reproduces the merge
- Killing a script mid-run leaves no partial sidecars (write to temp, move on success)
- A solo (single-track) recording and a two-track call both process through the same entry point
- VAD is enabled on every transcription run — no code path transcribes without it
- A transcript containing a repetition loop is flagged and blocked from cleanup, not passed through
- `prune.sh` never runs automatically

## 13. Explicitly out of scope

- Durable knowledge extraction — separate skills, run against `.clean.md`
- ML diarization — per-track capture covers the requirement
- Format conversion / `ffmpeg` — capture produces whisper's native working format
- **Editing journal notes to link the outputs.** Keep the blast radius small in v1; the pipeline
  writes sidecars and touches nothing you authored. Add note-linking later, once you trust it.
- Scheduling or file watching
