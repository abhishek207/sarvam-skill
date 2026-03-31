---
name: sarvam-ai
description: Sarvam AI expert guidance for building multilingual Indian language AI applications. Use when working with Sarvam APIs for speech-to-text, text-to-speech (Bulbul v3), translation, transliteration, language detection, chat completions, or building real-time voice agents for Indian languages. Trigger on: "sarvam", "bulbul", "saarika", "saaras", "mayura", "Indian language AI", "Hindi TTS", "Indian voice agent", "multilingual voice", "Indic language", "sarvamai".
metadata:
  version: 1.0.0
  docs:
    - "https://docs.sarvam.ai/api-reference-docs/introduction"
    - "https://docs.sarvam.ai/api-reference-docs/getting-started/models"
  pathPatterns:
    - "**/*sarvam*"
    - "**/*bulbul*"
    - "**/*saarika*"
  importPatterns:
    - "sarvamai"
    - "@sarvam"
---

# Sarvam AI

You are an expert in Sarvam AI — India's multilingual AI platform for Indian languages. It provides STT, TTS, translation, transliteration, language detection, chat completions, and real-time voice agent infrastructure.

## Setup & Authentication

**Install SDKs:**
```bash
# Python
pip install -U sarvamai

# JavaScript / Node.js
npm install sarvamai@latest
```

**Auth header** (all endpoints): `api-subscription-key: YOUR_API_KEY`

Get your key at: https://dashboard.sarvam.ai — new accounts get ₹1,000 free credits (never expire).

```python
from sarvamai import SarvamAI
client = SarvamAI(api_subscription_key="YOUR_KEY")
```

```typescript
import { SarvamAIClient } from "sarvamai";
const client = new SarvamAIClient({ apiSubscriptionKey: process.env.SARVAM_API_KEY });
```

**Base URL:** `https://api.sarvam.ai`

---

## Models Reference

| Model | Type | Notes |
|-------|------|-------|
| `sarvam-105b` | Chat LLM | 105B params, 128K context, flagship — currently free |
| `sarvam-30b` | Chat LLM | 30B params, 64K context, balanced — currently free |
| `bulbul:v3` | TTS | **Recommended** — 39+ voices, high quality, ₹30/10K chars |
| `bulbul:v2` | TTS | Older, ₹15/10K chars |
| `saaras:v3` | STT | State-of-the-art, 23 languages (22 Indic + English) |
| `saarika:v2.5` | STT | Standard ASR, transcribes in source language |
| `mayura:v1` | Translate | 12 languages, all modes + output scripts, auto-detect source |
| `sarvam-translate:v1` | Translate | All 22 scheduled Indian languages, formal mode only |

**Supported language codes (BCP-47):**
`hi-IN`, `bn-IN`, `ta-IN`, `te-IN`, `gu-IN`, `kn-IN`, `ml-IN`, `mr-IN`, `od-IN`, `pa-IN`, `en-IN`
Plus (translate only): `as-IN`, `brx-IN`, `doi-IN`, `kok-IN`, `ks-IN`, `mai-IN`, `mni-IN`, `ne-IN`, `sa-IN`, `sat-IN`, `sd-IN`, `ur-IN`

---

## Text-to-Speech (TTS) — Bulbul v3

**Endpoint:** `POST https://api.sarvam.ai/text-to-speech`

Bulbul v3 is the recommended TTS model with 39+ natural Indian voices, 11 languages, and up to 2500 characters per request.

```python
import base64, requests

response = requests.post(
    "https://api.sarvam.ai/text-to-speech",
    headers={"api-subscription-key": "YOUR_KEY"},
    json={
        "text": "नमस्ते, मैं आपकी सहायता कैसे कर सकता हूँ?",
        "target_language_code": "hi-IN",
        "model": "bulbul:v3",
        "speaker": "shubh",         # default voice
        "pace": 1.0,                # 0.5–2.0 for v3
        "temperature": 0.6,         # 0.01–2.0 for v3 (expressiveness)
        "speech_sample_rate": 24000, # 8000, 16000, 22050, 24000
        "output_audio_codec": "mp3"
    }
)
audio_bytes = base64.b64decode(response.json()["audios"][0])
with open("output.mp3", "wb") as f:
    f.write(audio_bytes)
```

### Bulbul v3 Speakers (39 voices)
| Voice | Characteristics |
|-------|----------------|
| `shubh` | Default male, warm and clear |
| `aditya` | Male, professional |
| `ritu` | Female, natural |
| `priya` | Female, energetic |
| `neha` | Female, clear |
| `rahul` | Male, deep |
| `pooja` | Female, warm |
| `rohan` | Male, young |
| `simran` | Female, expressive |
| `kavya` | Female, soft |

Full list (39 total): shubh, aditya, ritu, priya, neha, rahul, pooja, rohan, simran, kavya, amit, dev, ishita, shreya, ratan, varun, manan, sumit, roopa, kabir, aayan, ashutosh, advait, amelia, sophia, anand, tanya, tarun, sunny, mani, gokul, vijay, shruti, suhani, mohit, kavitha, rehan, soham, rupali

### v3 vs v2 parameter differences
| Parameter | Bulbul v3 | Bulbul v2 |
|-----------|-----------|-----------|
| `pace` | 0.5–2.0 | 0.3–3.0 |
| `temperature` | 0.01–2.0 ✓ | Not available |
| `pitch` | Not available | -0.75 to 0.75 |
| `loudness` | Not available | 0.3–3.0 |
| `text` max chars | 2500 | 1500 |
| `enable_preprocessing` | Not available | ✓ |
| `dict_id` (custom pronunciation) | ✓ | Not available |

---

## Speech-to-Text (STT)

**Endpoint:** `POST https://api.sarvam.ai/speech-to-text` (multipart/form-data)

**Supported formats:** WAV, MP3, AAC, AIFF, OGG, OPUS, FLAC, MP4/M4A, AMR, WMA, WebM, PCM (16kHz, `pcm_s16le`)
**Max duration:** 30 seconds per request (use Batch API for longer audio)

```python
with open("audio.wav", "rb") as f:
    response = requests.post(
        "https://api.sarvam.ai/speech-to-text",
        headers={"api-subscription-key": "YOUR_KEY"},
        files={"file": ("audio.wav", f, "audio/wav")},
        data={
            "model": "saaras:v3",
            "language_code": "hi-IN",  # or "unknown" for auto-detect
            "mode": "transcribe"        # saaras:v3 modes below
        }
    )
print(response.json()["transcript"])
```

### saaras:v3 modes
| Mode | Output |
|------|--------|
| `transcribe` | Transcription in source Indic language (default) |
| `translate` | Translates to English |
| `verbatim` | Word-for-word without fillers |
| `translit` | Romanized transliteration |
| `codemix` | Handles mixed language speech (Hinglish etc.) |

### Batch API (files up to 1 hour)
```python
# Submit batch job
response = requests.post(
    "https://api.sarvam.ai/speech-to-text/batch",
    headers={"api-subscription-key": "YOUR_KEY"},
    files=[("files", open("long_audio.mp3", "rb"))],
    data={"model": "saaras:v3", "language_code": "hi-IN"}
)
job_id = response.json()["job_id"]

# Poll for results
result = requests.get(
    f"https://api.sarvam.ai/speech-to-text/batch/{job_id}",
    headers={"api-subscription-key": "YOUR_KEY"}
)
```

---

## Translation

**Endpoint:** `POST https://api.sarvam.ai/translate`

```python
response = requests.post(
    "https://api.sarvam.ai/translate",
    headers={"api-subscription-key": "YOUR_KEY"},
    json={
        "input": "Hello, how are you?",
        "source_language_code": "en-IN",   # or "auto" for auto-detection
        "target_language_code": "hi-IN",
        "model": "mayura:v1",
        "mode": "formal",   # formal | modern-colloquial | classic-colloquial | code-mixed
        "output_script": None  # null | "roman" | "fully-native" | "spoken-form-in-native"
    }
)
print(response.json()["translated_text"])
```

**Character limits:** `mayura:v1` → 1000 chars; `sarvam-translate:v1` → 2000 chars

---

## Transliteration

**Endpoint:** `POST https://api.sarvam.ai/transliterate`

```python
response = requests.post(
    "https://api.sarvam.ai/transliterate",
    headers={"api-subscription-key": "YOUR_KEY"},
    json={
        "input": "namaste",
        "source_language_code": "en-IN",
        "target_language_code": "hi-IN",
        "spoken_form": True   # converts to natural spoken form
    }
)
print(response.json()["transliterated_text"])  # "नमस्ते"
```

---

## Language Detection

**Endpoint:** `POST https://api.sarvam.ai/text-lid`

```python
response = requests.post(
    "https://api.sarvam.ai/text-lid",
    headers={"api-subscription-key": "YOUR_KEY"},
    json={"input": "मैं ठीक हूँ"}  # max 1000 chars
)
# Returns: { "language_code": "hi-IN", "script_code": "Deva" }
```

---

## Chat Completions (OpenAI-compatible)

**Endpoint:** `POST https://api.sarvam.ai/v1/chat/completions`

Also accepts `Authorization: Bearer sk_xxx` header (in addition to `api-subscription-key`).

```python
response = requests.post(
    "https://api.sarvam.ai/v1/chat/completions",
    headers={"api-subscription-key": "YOUR_KEY"},
    json={
        "model": "sarvam-105b",
        "messages": [
            {"role": "system", "content": "You are a helpful assistant that responds in Hindi."},
            {"role": "user", "content": "भारत की राजधानी क्या है?"}
        ],
        "temperature": 0.7,
        "stream": False,
        "wiki_grounding": True   # grounds response in Wikipedia (Indic knowledge)
    }
)
```

**Streaming:**
```python
import json

with requests.post(
    "https://api.sarvam.ai/v1/chat/completions",
    headers={"api-subscription-key": "YOUR_KEY"},
    json={"model": "sarvam-105b", "messages": [...], "stream": True},
    stream=True
) as resp:
    for line in resp.iter_lines():
        if line.startswith(b"data: "):
            chunk = json.loads(line[6:])
            if chunk.get("choices"):
                print(chunk["choices"][0]["delta"].get("content", ""), end="")
```

---

## Building AI Voice Agents with Bulbul v3

Bulbul v3 is the backbone for production-grade Indian-language voice agents. Sarvam provides first-class integrations with **LiveKit** and **Pipecat** for real-time voice pipelines.

### Architecture Overview

```
User Audio (mic)
    ↓
STT: Saaras v3 WebSocket (real-time streaming, 8kHz/16kHz)
    ↓
LLM: sarvam-105b / sarvam-30b (with Hindi/Indic system prompt)
    ↓
TTS: Bulbul v3 WebSocket (streaming audio output)
    ↓
User Speaker
```

### Option 1: LiveKit Voice Agent

LiveKit is the recommended option for production voice agents with Sarvam.

```bash
pip install livekit-agents[sarvam,openai,silero]
```

```python
from livekit.agents import AutoSubscribe, JobContext, WorkerOptions, cli, llm
from livekit.agents.voice_assistant import VoiceAssistant
from livekit.plugins import openai, silero
from livekit.plugins.sarvam import STT, TTS

async def entrypoint(ctx: JobContext):
    initial_ctx = llm.ChatContext().append(
        role="system",
        text="You are a helpful voice assistant that speaks Hindi fluently. "
             "Keep responses concise for voice.",
    )

    await ctx.connect(auto_subscribe=AutoSubscribe.AUDIO_ONLY)

    assistant = VoiceAssistant(
        vad=silero.VAD.load(),
        stt=STT(
            model="saaras:v3",
            language="hi-IN",
            api_key="YOUR_SARVAM_KEY",
        ),
        llm=openai.LLM(model="sarvam-105b", base_url="https://api.sarvam.ai/v1"),
        tts=TTS(
            model="bulbul:v3",
            speaker="shubh",          # choose from 39 voices
            language_code="hi-IN",
            pace=1.0,
            temperature=0.6,
            api_key="YOUR_SARVAM_KEY",
        ),
        chat_ctx=initial_ctx,
    )
    assistant.start(ctx.room)
    await assistant.say("नमस्ते! मैं आपकी किस प्रकार सहायता कर सकता हूँ?", allow_interruptions=True)

if __name__ == "__main__":
    cli.run_app(WorkerOptions(entrypoint_fnc=entrypoint))
```

**Environment variables for LiveKit:**
```bash
LIVEKIT_URL=wss://your-project-xxxxx.livekit.cloud
LIVEKIT_API_KEY=your_livekit_api_key
LIVEKIT_API_SECRET=your_livekit_api_secret
SARVAM_API_KEY=your_sarvam_api_key
```

### Option 2: Pipecat Voice Agent

Pipecat is a framework-agnostic pipeline for real-time voice agents.

```bash
pip install pipecat-ai[sarvam,daily]
```

```python
import asyncio
from pipecat.pipeline.pipeline import Pipeline
from pipecat.pipeline.runner import PipelineRunner
from pipecat.services.sarvam import SarvamSTTService, SarvamTTSService
from pipecat.transports.services.daily import DailyTransport, DailyParams

async def main():
    transport = DailyTransport("YOUR_DAILY_ROOM_URL", None, "Voice Agent",
        DailyParams(audio_out_enabled=True, vad_enabled=True))

    stt = SarvamSTTService(
        api_key="YOUR_SARVAM_KEY",
        model="saaras:v3",
        language="hi-IN",
    )

    tts = SarvamTTSService(
        api_key="YOUR_SARVAM_KEY",
        model="bulbul:v3",
        speaker="priya",        # female voice for this agent
        language_code="hi-IN",
        sample_rate=16000,      # use 8000 for telephony
    )

    pipeline = Pipeline([transport.input(), stt, your_llm, tts, transport.output()])
    runner = PipelineRunner()
    await runner.run(pipeline)

asyncio.run(main())
```

### Option 3: Direct WebSocket (Custom Integration)

For full control, use Sarvam's WebSocket APIs directly.

**STT WebSocket:**
```python
import asyncio, websockets, json

async def stream_stt():
    uri = "wss://api.sarvam.ai/speech-to-text-streaming"
    headers = {"api-subscription-key": "YOUR_KEY"}

    async with websockets.connect(uri, additional_headers=headers) as ws:
        # Send config
        await ws.send(json.dumps({
            "model": "saaras:v3",
            "language_code": "hi-IN",
            "flush_signal": True  # enables flush for low latency (min_endpointing_delay: 70ms)
        }))

        # Stream audio chunks (PCM 16kHz or 8kHz)
        with open("audio.raw", "rb") as f:
            while chunk := f.read(3200):  # 100ms of 16kHz PCM
                await ws.send(chunk)

        # Send end signal
        await ws.send(json.dumps({"type": "end"}))

        # Receive transcripts
        async for message in ws:
            data = json.loads(message)
            if data.get("type") == "transcript":
                print(data["transcript"])

asyncio.run(stream_stt())
```

**TTS WebSocket (streaming audio out):**
```python
async def stream_tts(text: str):
    uri = "wss://api.sarvam.ai/text-to-speech-streaming"
    headers = {"api-subscription-key": "YOUR_KEY"}

    async with websockets.connect(uri, additional_headers=headers) as ws:
        await ws.send(json.dumps({
            "text": text,
            "model": "bulbul:v3",
            "target_language_code": "hi-IN",
            "speaker": "shubh",
            "speech_sample_rate": 8000,         # use 8kHz for telephony/WebRTC
            "output_audio_codec": "mulaw"        # mulaw for telephony
        }))

        audio_chunks = []
        async for message in ws:
            if isinstance(message, bytes):
                audio_chunks.append(message)
            elif isinstance(message, str):
                data = json.loads(message)
                if data.get("type") == "end":
                    break

        return b"".join(audio_chunks)
```

### Voice Agent Best Practices

**For telephony (PSTN/SIP):**
- Use `speech_sample_rate: 8000` and `output_audio_codec: "mulaw"` or `"alaw"`
- STT: send PCM at 8kHz with `input_audio_codec: "pcm_raw"`
- Use `flush_signal: True` in STT WebSocket for 70ms endpointing latency

**For WebRTC/browser:**
- Use `speech_sample_rate: 16000` or `24000`
- `output_audio_codec: "opus"` for best compression over WebRTC
- LiveKit handles codec negotiation automatically

**Language routing (multi-language agent):**
```python
# Auto-detect language from first utterance, then lock
lid_response = requests.post(
    "https://api.sarvam.ai/text-lid",
    headers={"api-subscription-key": "YOUR_KEY"},
    json={"input": user_transcript}
)
detected_lang = lid_response.json()["language_code"]

# Use detected language for TTS response
tts_response = requests.post(
    "https://api.sarvam.ai/text-to-speech",
    headers={"api-subscription-key": "YOUR_KEY"},
    json={
        "text": llm_response,
        "target_language_code": detected_lang,
        "model": "bulbul:v3",
        "speaker": "shubh"
    }
)
```

**Choosing Bulbul v3 voice for your use case:**
| Use Case | Recommended Voice |
|----------|------------------|
| Customer support (neutral) | `shubh` (default) |
| Customer support (female) | `priya` or `ritu` |
| Banking / finance | `aditya` or `rahul` (formal) |
| E-commerce / retail | `neha` or `pooja` |
| Healthcare | `kavya` (soft) or `shruti` |
| Education / tutoring | `rohan` or `ishita` |
| Infotainment / news | `varun` or `tanya` |

---

## Full End-to-End Voice Pipeline (Python)

```python
import asyncio, base64, requests

SARVAM_KEY = "YOUR_SARVAM_KEY"
HEADERS = {"api-subscription-key": SARVAM_KEY}

def transcribe_audio(audio_path: str) -> str:
    """STT: audio file → Hindi transcript"""
    with open(audio_path, "rb") as f:
        resp = requests.post(
            "https://api.sarvam.ai/speech-to-text",
            headers=HEADERS,
            files={"file": (audio_path, f, "audio/wav")},
            data={"model": "saaras:v3", "language_code": "hi-IN"}
        )
    return resp.json()["transcript"]

def get_llm_response(user_text: str, history: list) -> str:
    """LLM: text → response in Hindi"""
    messages = [
        {"role": "system", "content": "आप एक सहायक हैं। हमेशा हिंदी में जवाब दें।"},
        *history,
        {"role": "user", "content": user_text}
    ]
    resp = requests.post(
        "https://api.sarvam.ai/v1/chat/completions",
        headers=HEADERS,
        json={"model": "sarvam-105b", "messages": messages, "temperature": 0.7}
    )
    return resp.json()["choices"][0]["message"]["content"]

def synthesize_speech(text: str, output_path: str):
    """TTS: Hindi text → Bulbul v3 audio"""
    resp = requests.post(
        "https://api.sarvam.ai/text-to-speech",
        headers=HEADERS,
        json={
            "text": text[:2500],   # v3 limit
            "target_language_code": "hi-IN",
            "model": "bulbul:v3",
            "speaker": "shubh",
            "pace": 1.0,
            "temperature": 0.6,
            "speech_sample_rate": 24000,
            "output_audio_codec": "mp3"
        }
    )
    audio = base64.b64decode(resp.json()["audios"][0])
    with open(output_path, "wb") as f:
        f.write(audio)

# Voice agent turn loop
async def voice_agent():
    history = []
    while True:
        # 1. STT
        transcript = transcribe_audio("user_turn.wav")
        print(f"User: {transcript}")

        # 2. LLM
        response = get_llm_response(transcript, history)
        print(f"Agent: {response}")

        # 3. TTS
        synthesize_speech(response, "agent_turn.mp3")

        # Update history
        history += [
            {"role": "user", "content": transcript},
            {"role": "assistant", "content": response}
        ]

asyncio.run(voice_agent())
```

---

## Error Handling

All endpoints return standard HTTP codes. Common errors:

| Code | Error | Fix |
|------|-------|-----|
| 400 | `invalid_request_error` | Check required params and formats |
| 403 | `invalid_api_key_error` | Verify `api-subscription-key` header |
| 422 | `unprocessable_entity_error` | Audio too long / unsupported format |
| 429 | `rate_limit_exceeded_error` | Back off and retry; upgrade plan |
| 402 | `insufficient_quota_error` | Recharge credits at dashboard.sarvam.ai |
| 503 | `internal_server_error` | Retry with exponential backoff |

```python
import time

def call_with_retry(fn, *args, retries=3, **kwargs):
    for i in range(retries):
        resp = fn(*args, **kwargs)
        if resp.status_code == 429:
            time.sleep(2 ** i)
            continue
        resp.raise_for_status()
        return resp
    raise Exception("Max retries exceeded")
```

---

## Pricing Summary

| API | Rate |
|-----|------|
| `sarvam-105b` Chat | Free |
| `sarvam-30b` Chat | Free |
| Bulbul v3 TTS | ₹30 / 10K chars |
| Bulbul v2 TTS | ₹15 / 10K chars |
| STT (standard) | ₹30 / hour |
| STT + Diarization | ₹45 / hour |
| STT + Translation | ₹30 / hour |
| Sarvam Translate v1 | ₹20 / 10K chars |
| Mayura v1 Translation | ₹20 / 10K chars |
| Transliteration | ₹20 / 10K chars |
| Language Detection | ₹3.50 / 10K chars |
| Sarvam Vision | ₹1.50 / page |
| **Free credits on signup** | ₹1,000 (never expire) |

## Rate Limits by Plan

| Plan | Rate Limit | Monthly Cost |
|------|-----------|-------------|
| Starter | 60 req/min | Pay-as-you-go |
| Pro | 200 req/min | ₹10,000 + ₹1,000 bonus |
| Business | 1,000 req/min | ₹50,000 + ₹7,500 bonus |
| Enterprise | Custom | Custom |

---

## Quick Reference

```bash
# Environment variable
SARVAM_API_KEY=your_api_key_here
```

```python
# Minimal TTS (Bulbul v3)
import requests, base64
r = requests.post("https://api.sarvam.ai/text-to-speech",
    headers={"api-subscription-key": "YOUR_KEY"},
    json={"text": "नमस्ते", "target_language_code": "hi-IN",
          "model": "bulbul:v3", "speaker": "shubh"})
audio = base64.b64decode(r.json()["audios"][0])

# Minimal STT
r = requests.post("https://api.sarvam.ai/speech-to-text",
    headers={"api-subscription-key": "YOUR_KEY"},
    files={"file": open("audio.wav","rb")},
    data={"model": "saaras:v3", "language_code": "hi-IN"})
print(r.json()["transcript"])
```
