---
name: agnes-api
description: Call the Agnes AI API (OpenAI-compatible) for text, image, video, and multimodal generation. Use when the user wants to integrate with, call, or debug requests to Agnes AI, apihub.agnes-ai.com, or an `agnes-*` model.
---

Agnes AI exposes an OpenAI-compatible API. Existing OpenAI client code usually only needs three changes: base URL, API key, model name.

## Connection

- Base URL (international, primary): `https://apihub.agnes-ai.com/v1`
- Alternate, only for network/DNS/TLS/connect-timeout failures: `https://apihub.agnes-ai.cn/v1`
- China service: `https://api.agnes-ai.cn/v1`
- Auth header: `Authorization: Bearer $AGNES_API_KEY`
- Keys come from https://platform.agnes-ai.com/. Read the key from the environment. Never hardcode it, commit it, put it in front-end code, or echo it in output. If it leaks, tell the user to reset it in the Agnes console.

## Chat completions

```bash
curl https://apihub.agnes-ai.com/v1/chat/completions \
  -H "Authorization: Bearer $AGNES_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "agnes-2.5-flash",
    "messages": [{"role": "user", "content": "Hello"}]
  }'
```

With the OpenAI SDK, set `base_url="https://apihub.agnes-ai.com/v1"` and `api_key` from the environment; the rest of the call is unchanged.

## Image generation

```bash
curl https://apihub.agnes-ai.com/v1/images/generations \
  -H "Authorization: Bearer $AGNES_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"model": "agnes-image-2.1-flash", "prompt": "A floating city at sunrise", "size": "1024x768"}'
```

Output is a URL or Base64. Editing inputs go in `extra_body.image` (shape not documented). Prefer public image URLs over Base64 to avoid 413.

## Video generation (asynchronous)

```bash
curl -X POST https://apihub.agnes-ai.com/v1/videos \
  -H "Authorization: Bearer $AGNES_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"model": "agnes-video-v2.0", "prompt": "A cat walking on a beach at sunset", "height": 768, "width": 1152, "num_frames": 121, "frame_rate": 24}'
```

Poll the result with the returned `video_id`: `GET https://apihub.agnes-ai.com/agnesapi?video_id=<VIDEO_ID>`. Do not use `task_id` unless a legacy workflow requires it. The create-response shape and status values are not documented.

## Models

The two official READMEs (AgnesAI-Labs/AgnesAI-Models and AgnesAI-Labs/skills) disagree, so verify against the account before relying on a name.

- Text/VLM: `agnes-2.5-flash`, `agnes-2.0-flash`
- Image: `agnes-image-2.0-flash`, `agnes-image-2.1-flash` (Models repo); `agnes-image-2.5-flash` (skills repo)
- Video: `agnes-video-v2.0` (Models repo); `agnes-video-2.5`, `agnes-video-2.5-flash` (skills repo)

A wrong model name can surface as 404 or 503.

## Errors

Retry with backoff on 408, 429, 500, 502, 503, 504, 520, 522, 524. Otherwise fix the request:

- 401: missing, bad, or expired key
- 402: balance or Token Plan quota exhausted
- 403: model not enabled, or policy/IP/region block
- 404: wrong path, doubled `/v1`, or wrong model
- 413 / 415: body too large, or format not PNG/JPG/JPEG/WEBP
- 422: bad values (size, aspect ratio, frame rate, inaccessible image URL)
- 429: RPM exceeded (default tier is roughly 20 RPM text, 1 RPM video)

The JSON error body format is not documented.

## Not yet documented

Full image and video request/response schemas, video status values, and `MODEL_CATALOG.md`. Official docs: https://agnes-ai.com/doc/overview. Do not guess these; check the docs or ask the user.
