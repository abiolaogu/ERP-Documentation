# ERP-Assistant Voice Service Specification

## 1. Overview

The voice-service provides speech-to-text (STT) and text-to-speech (TTS) capabilities for ERP-Assistant, enabling hands-free interaction with the ERP platform. Built on Python/FastAPI, it integrates OpenAI Whisper for transcription and ElevenLabs/Coqui for synthesis, communicating with clients via WebSocket for real-time streaming.

### Voice Pipeline Architecture

```mermaid
flowchart LR
    subgraph "Client"
        MIC["Microphone<br/>PCM 16-bit 16kHz"]
        SPK["Speaker<br/>PCM 16-bit 24kHz"]
    end

    subgraph "voice-service"
        WS["WebSocket<br/>Handler"]
        WAKE["Wake Word<br/>Detector"]
        VAD["Voice Activity<br/>Detection"]
        STT["Whisper<br/>Large-v3"]
        TTS_ENG["ElevenLabs /<br/>Coqui TTS"]
        BUFF["Audio Buffer<br/>Manager"]
    end

    subgraph "assistant-core"
        NLP["NLP Pipeline"]
    end

    MIC -->|binary frames| WS
    WS --> WAKE --> VAD --> STT
    STT -->|transcript text| NLP
    NLP -->|response text| TTS_ENG
    TTS_ENG -->|audio stream| WS
    WS -->|binary frames| SPK
```

## 2. Current Implementation

The voice-service exposes a FastAPI application with health check and listing endpoints:

```python
# services/voice-service/main.py
from fastapi import FastAPI, Header, HTTPException

app = FastAPI(title="ERP-Assistant voice-service")

@app.get('/healthz')
def healthz():
    return {"status": "healthy", "module": "ERP-Assistant", "service": "voice-service"}

@app.get('/v1/voice')
def list_items(x_tenant_id: str | None = Header(default=None)):
    if not x_tenant_id:
        raise HTTPException(status_code=400, detail='missing X-Tenant-ID')
    return {"items": [], "event_topic": "erp.assistant.voice.listed"}
```

## 3. STT Specification (Whisper)

### Model Configuration

| Parameter | Value | Notes |
|-----------|-------|-------|
| Model | Whisper Large-v3 | Best accuracy, GPU recommended |
| Fallback model | Whisper Medium | CPU-friendly, slightly lower accuracy |
| Input format | PCM 16-bit, 16kHz, mono | Standard microphone format |
| Language | Auto-detect (155 languages) | Override via config |
| Beam size | 5 | Balance accuracy vs speed |
| Word timestamps | Enabled | For real-time streaming |
| VAD filter | Enabled | Skip silence, reduce processing |

### Streaming Transcription Protocol

```mermaid
sequenceDiagram
    participant C as Client
    participant VS as voice-service
    participant W as Whisper

    C->>VS: WS Connect (/v1/voice/stream)
    C->>VS: {"action": "start_listening"}

    loop Audio Stream
        C->>VS: [binary: PCM audio chunk, 100ms]
        VS->>VS: Buffer audio (min 500ms)
        VS->>W: Process buffered audio
        W-->>VS: Partial transcript
        VS-->>C: {"type": "transcript", "text": "What's my", "is_final": false}
    end

    C->>VS: {"action": "stop_listening"}
    VS->>W: Process remaining buffer
    W-->>VS: Final transcript
    VS-->>C: {"type": "transcript", "text": "What's my revenue?", "is_final": true, "confidence": 0.97}
```

### Accuracy Benchmarks

| Language | WER (Word Error Rate) | Notes |
|----------|----------------------|-------|
| English | 3.5% | Large-v3 on LibriSpeech |
| Spanish | 5.2% | Business domain |
| French | 4.8% | Business domain |
| German | 5.5% | Business domain |
| Chinese (Mandarin) | 6.1% | Business domain |

## 4. TTS Specification

### ElevenLabs (Cloud)

| Parameter | Value |
|-----------|-------|
| API | Streaming speech synthesis |
| Voices | Configurable per user preference |
| Output format | PCM 16-bit, 24kHz, mono |
| Latency (first byte) | ~300ms |
| Quality (MOS) | 4.5/5 |
| Cost | $0.30 per 1,000 characters |
| Streaming | Chunked transfer, sentence-level |

### Coqui TTS (Self-Hosted)

| Parameter | Value |
|-----------|-------|
| Model | XTTS v2 |
| Voices | Cloneable from sample audio |
| Output format | PCM 16-bit, 24kHz, mono |
| Latency (first byte) | ~150ms |
| Quality (MOS) | 3.8/5 |
| Cost | Free (self-hosted) |
| GPU | Optional (faster with CUDA) |

### TTS Selection Logic

```mermaid
flowchart TB
    REQ["TTS Request"] --> CHECK{"ElevenLabs<br/>API Key<br/>Configured?"}
    CHECK -->|Yes| EL_CHECK{"API Quota<br/>Remaining?"}
    EL_CHECK -->|Yes| ELEVEN["ElevenLabs<br/>(Cloud)"]
    EL_CHECK -->|No| COQUI["Coqui TTS<br/>(Self-Hosted)"]
    CHECK -->|No| COQUI
    COQUI --> OUTPUT["Audio Output"]
    ELEVEN --> OUTPUT
```

## 5. Wake Word Detection

### Configuration

| Parameter | Value |
|-----------|-------|
| Default wake word | "Hey Assistant" |
| Customizable | Yes, per-tenant |
| Engine | Porcupine / openWakeWord |
| False positive rate | < 1 per hour |
| Detection latency | < 100ms |

### Wake Word Flow

```mermaid
stateDiagram-v2
    [*] --> Idle: Service started
    Idle --> WakeWordDetected: Wake word heard
    WakeWordDetected --> Listening: Start recording
    Listening --> Processing: Silence detected / stop command
    Processing --> Responding: NLP response ready
    Responding --> Listening: Follow-up expected
    Responding --> Idle: Conversation ended (timeout 10s)
    Listening --> Idle: Cancel command / timeout 30s
```

## 6. Audio Processing Pipeline

### Input Processing

```mermaid
flowchart LR
    RAW["Raw Audio<br/>(any format)"]
    RESAMPLE["Resample<br/>to 16kHz mono"]
    DENOISE["Noise<br/>Reduction"]
    AGC["Auto Gain<br/>Control"]
    VAD["Voice Activity<br/>Detection"]
    BUFFER["Audio Buffer<br/>(ring buffer, 30s max)"]

    RAW --> RESAMPLE --> DENOISE --> AGC --> VAD --> BUFFER
```

### Output Processing

```mermaid
flowchart LR
    TEXT["Response Text"] --> SSML["SSML Markup<br/>(pauses, emphasis)"]
    SSML --> TTS["TTS Engine"]
    TTS --> NORM["Audio<br/>Normalization"]
    NORM --> STREAM["WebSocket<br/>Stream"]
```

## 7. WebSocket Protocol

### Client Messages

| Message | Type | Payload |
|---------|------|---------|
| Start listening | text | `{"action": "start_listening"}` |
| Stop listening | text | `{"action": "stop_listening"}` |
| Cancel | text | `{"action": "cancel"}` |
| Audio data | binary | PCM 16-bit 16kHz mono |
| Set language | text | `{"action": "set_language", "language": "en"}` |
| Set voice | text | `{"action": "set_voice", "voice": "professional-female"}` |

### Server Messages

| Message | Type | Payload |
|---------|------|---------|
| Transcript (partial) | text | `{"type": "transcript", "text": "...", "is_final": false}` |
| Transcript (final) | text | `{"type": "transcript", "text": "...", "is_final": true, "confidence": 0.97}` |
| AI response | text | `{"type": "response", "text": "...", "audio_url": "..."}` |
| Audio stream | binary | PCM 16-bit 24kHz mono |
| Error | text | `{"type": "error", "message": "..."}` |
| Status | text | `{"type": "status", "state": "listening|processing|responding|idle"}` |

## 8. Resource Requirements

| Component | CPU | Memory | GPU | Storage |
|-----------|-----|--------|-----|---------|
| Whisper Large-v3 | 4 cores | 2GB | Recommended | 3GB model |
| Whisper Medium | 2 cores | 1GB | Optional | 1.5GB model |
| ElevenLabs client | 0.5 cores | 100MB | No | - |
| Coqui XTTS v2 | 2 cores | 1GB | Recommended | 2GB model |
| FastAPI server | 1 core | 256MB | No | - |

## 9. Error Handling

| Error | Cause | Recovery |
|-------|-------|----------|
| STT timeout | No speech detected for 30s | Return to idle, notify client |
| STT low confidence | Noisy audio, accent | Request repeat, fallback to text |
| TTS API failure | ElevenLabs API error | Fallback to Coqui TTS |
| WebSocket disconnect | Network issue | Client auto-reconnect with exponential backoff |
| Model load failure | OOM or missing files | Start without voice, log error |
