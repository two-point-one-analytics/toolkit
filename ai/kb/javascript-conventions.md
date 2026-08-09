# JavaScript Conventions

## Naming And Wire Boundaries

Summary: Use JS-native names internally and reserve external-case names for system boundaries.

Context: These conventions apply to JavaScript and TypeScript across projects. Repo-local runtime constraints, formatters, and API contracts still take precedence when they are more specific.

Learning: Keep values and behavior in `camelCase`, types and constructable things in `PascalCase`, true constants in `SCREAMING_SNAKE_CASE`, wire-format fields in `snake_case`, and non-JS identifiers in `kebab-case`.

| Convention | Use For | Examples |
|---|---|---|
| `PascalCase` | Types, interfaces, classes, enums, React components, generic type parameters | `PropertyPayload`, `HttpClient`, `SessionState`, `LogLevel`, `TInput` |
| `camelCase` | Variables, functions, methods, parameters, internal object keys | `dailyRate`, `getProperty()`, `obj.doThing()`, `{ firstName: 'X' }` |
| `SCREAMING_SNAKE_CASE` | True constants, build-time defines, env-like config | `MAX_RETRIES`, `API_BASE_URL`, `__BUILD_VERSION__` |
| `snake_case` | Wire format only, never internal local names | data layer keys, JSON API fields, DB columns, URL query params |
| `kebab-case` | Identifiers that cannot be JS variables | `to-number.ts`, CSS classes, `data-property-id`, npm packages, CLI flags |

Inside function bodies, use `camelCase`. Convert to wire-format casing only where data crosses a boundary, usually in the returned object, data layer push, API request body, DB write payload, or query string construction.

```typescript
function getProperty(): PropertyPayload {
  const dailyRate = toNumber(...);
  const isStrikethrough = toBoolean(...);

  return {
    daily_rate_metric: dailyRate,
    is_strikethrough_rate: isStrikethrough,
  };
}
```

Use a `Payload` suffix for types whose values flow directly into wire format and therefore may intentionally contain `snake_case` keys, such as `PropertyPayload`, `SessionPayload`, or `EcommercePayload`. Use plain internal names such as `Config` or `LogLevel` for shapes that stay inside the program and use `camelCase` keys.

Quick model: type or constructable thing -> `PascalCase`; value or behavior -> `camelCase`; true constant -> `SCREAMING_SNAKE_CASE`; system boundary -> `snake_case` at the edge; invalid JS identifier -> `kebab-case`.

## Formatting

Where a repo configures Prettier, it is the formatting source of truth. Baseline:

- 2-space indentation
- Single quotes
- Semicolons required
- Trailing commas where valid (ES5: objects and arrays)
- Unquoted object keys when possible (quote only when required)
- ~100-character line width

## Script Structure

- Order every script's sections `Config` -> `Functions` -> `Main`, with banner headers:

```js
// ── Config ─────────────────────────────────────
// ── Functions ──────────────────────────────────
// ── Main ───────────────────────────────────────
```

- Define standalone helpers as named `function` declarations. Reserve arrow functions for inline callbacks (`.map()`, `.filter()`, event handlers).
- Never convert a named `function` into an arrow function assigned to `const`/`var`.
- Keep blank lines between logical blocks; avoid excessive gaps.

## Comments

- Explain *why*, not *what*: business logic, non-obvious decisions, platform constraints, workarounds.
- Remove comments that merely restate the code.
- Remove commented-out code unless it serves as a documented config toggle (e.g., alternate IDs).

## Conciseness And Modern Syntax

Prefer idiomatic, concise patterns when the runtime supports them:

- Iterator methods (`.map()`, `.filter()`, `.reduce()`, `.forEach()`) over manual `for` loops.
- `.includes(x)` over `indexOf(x) > -1`.
- `Math.round(x * 10**n) / 10**n` over `parseFloat(x.toFixed(n))` for rounding — returns a number directly, no string roundtrip.
- Optional chaining `?.`, nullish coalescing `??`, destructuring, spread, template literals, `Object.fromEntries()`.
- Collapse unnecessary intermediate variables; remove dead code, unused variables, and unreachable branches.
- Omit explicitly-assigned `undefined` values from object literals.
- Descriptive names that reflect purpose; never rename variables that match external API field names (e.g., data layer keys, JSON API fields).

## Environment-Specific Constraints

Runtime sandboxes can forbid modern syntax, so these conventions apply only to the extent the target environment supports them. Repo-local docs and config are authoritative when more specific — a repo targeting a constrained sandbox should document which patterns are allowed per environment. Tag-manager template sandboxes are a common example, often disallowing template literals, destructuring, spread, `?.`, `??`, and `.includes()`.
