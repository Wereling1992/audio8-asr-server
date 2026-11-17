# Audio8 ASR Server

Lightweight **CPU-only** Automatic Speech Recognition service powered by [Audio8-ASR-0.1B](https://huggingface.co/Edge0/Audio8-ASR-0.1B).

Built with FastAPI + ONNX Runtime. Drop-in friendly for apps that expect an OpenAI-style transcription API.

**Docker Hub:** [wereling1992/audio8-asr-server](https://hub.docker.com/r/wereling1992/audio8-asr-server)

## Features

- Simple REST API: `POST /asr`
- **OpenAI-compatible**: `POST /v1/audio/transcriptions`
- API key auth (`x-api-key` or `Authorization: Bearer`)
- Hotword boosting (no fine-tuning)
- Precision options: int8 (default) / int4 / fp32
- Multilingual: Chinese, English, Cantonese, French, German, Japanese, Korean
- Web UI at http://localhost:8000/
- **CPU-only** — no GPU required

## Quick Start

### Docker (recommended)

```bash
docker run -d \
  -p 8000:8000 \
  -e AUDIO8_API_KEY=MySuperSafeApiKey \
  -v audio8-models:/models \
  --name audio8-asr \
  wereling1992/audio8-asr-server:latest
```

First start downloads the model (~3 GB) into the volume. Later starts reuse it.

#### Health check

```bash
curl http://localhost:8000/health
```

#### Transcribe (native API)

```bash
curl -X POST http://localhost:8000/asr \
  -H "x-api-key: MySuperSafeApiKey" \
  -F "file=@your.wav" \
  -F "hotwords=Audio8,Edge0"
```

#### OpenAI-compatible API

```bash
curl -X POST http://localhost:8000/v1/audio/transcriptions \
  -H "Authorization: Bearer MySuperSafeApiKey" \
  -F "file=@your.wav" \
  -F "model=audio8"
```

```python
from openai import OpenAI

client = OpenAI(base_url="http://localhost:8000/v1", api_key="MySuperSafeApiKey")
with open("your.wav", "rb") as f:
    r = client.audio.transcriptions.create(model="audio8", file=f)
print(r.text)
```

Web UI: http://localhost:8000/

### Docker Compose / Build from source

```bash
git clone https://github.com/Wereling1992/audio8-asr-server.git
cd audio8-asr-server
docker compose up -d --build
```

## Environment Variables

| Variable               | Default  | Description                    |
|------------------------|----------|--------------------------------|
| AUDIO8_API_KEY         | changeme | API key                        |
| AUDIO8_REQUIRE_API_KEY | 1        | Enforce API key (0 to disable) |
| ASR_CACHE_PRECISION    | int8     | int8 / int4 / fp32             |
| ASR_AUDIO_PRECISION    | int8     | int8 / fp32                    |
| ASR_BACKEND            | auto     | auto / onnx_cache / onnx       |
| FORCE_DOWNLOAD         | 0        | Force re-download model        |
| PORT                   | 8000     | Listen port                    |

## API

### `GET /health`

No auth. Returns service status.

### `POST /asr`

Header: `x-api-key: <key>`

| Form field       | Type   | Description                              |
|------------------|--------|------------------------------------------|
| file / audio     | file   | Audio file (wav/mp3/flac/ogg/m4a…)       |
| hotwords         | string | Optional comma-separated boost terms     |
| max_new_tokens   | int    | Generation limit (default 128)           |
| cache_precision  | string | Override: int8 / int4 / fp32             |
| audio_precision  | string | Override: int8 / fp32                    |

### `POST /v1/audio/transcriptions`

OpenAI-compatible. Auth: `Authorization: Bearer <key>` or `x-api-key: <key>`.

Supported `response_format`: `json` (default), `text`, `verbose_json`.

## Resource Recommendation

- **RAM**: ~1.5–2.5 GB peak (int8)
- **Disk**: ~3.5 GB (model + runtime)
- **GPU**: not required

## License

Model weights & original ONNX runtime: **CC BY-NC 4.0** (non-commercial).

© Edge0 / AutoArk-AI — https://huggingface.co/Edge0/Audio8-ASR-0.1B

This image packages the official ONNX runtime for convenient deployment.

## Credits

- [Edge0 / AutoArk-AI](https://edge0.ai) for Audio8-ASR-0.1B
- Official package: [Edge0/Audio8-ASR-0.1B-onnx-runtime](https://huggingface.co/Edge0/Audio8-ASR-0.1B-onnx-runtime)
- Inspired by [DaDao-Labs/SenseVoice-ASR-Server](https://github.com/DaDao-Labs/SenseVoice-ASR-Server)
