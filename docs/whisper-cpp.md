# Install

```bash
brew install whisper-cpp ffmpeg
```

# Download Model
[Model List](https://huggingface.co/ggerganov/whisper.cpp)

```bash
mkdir -p ~/models/whisper && cd ~/models/whisper
curl -L -o ggml-large-v3-turbo-q8_0.bin \
  'https://huggingface.co/ggerganov/whisper.cpp/resolve/main/ggml-large-v3-turbo-q8_0.bin?download=true'
```




