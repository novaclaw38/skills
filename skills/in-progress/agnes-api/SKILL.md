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

Create a task with `POST /v1/videos`, then poll with the returned `video_id`. `id` and `task_id` identify the task, but `video_id` is what retrieval uses.

```bash
curl -sS -X POST https://apihub.agnes-ai.com/v1/videos \
  -H "Authorization: Bearer $AGNES_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"model": "agnes-video-2.5", "prompt": "A futuristic city street after rain, neon reflections, a silver car passing slowly", "seconds": "5", "mode": "text", "size": "720P", "aspect_ratio": "16:9"}'

curl -sS "https://apihub.agnes-ai.com/agnesapi?video_id=VIDEO_ID&model_name=agnes-video-2.5" \
  -H "Authorization: Bearer $AGNES_API_KEY"
```

Always use the query form with `model_name`, for every mode. Poll every 1 to 2 seconds until `status` is `completed` or `failed`, then use `url`.

`agnes-video-2.5` parameters:

- `model` (required), `prompt` (required), `mode` (required): `text`, `keyframe`, or `reference`
- `seconds`: string, `"4"` to `"12"`, default `"5"`
- `size`: `"720P"`, `"1080P"`, `"1K"`, `"2K"`
- `aspect_ratio`: default `16:9`
- `seed`: integer. `n`: only `1` is supported.
- keyframe mode: `first_frame` and/or `last_frame` (image URLs)
- reference mode: `images` (URLs), `audios` (URLs), `videos` (objects); refer to them in the prompt as `<Picture N>`, `<Audio N>`, `<Video N>`

Pricing for `agnes-video-2.5`: 720P $0.025/s, 1080P and 1K $0.040/s, 2K $0.055/s. Tell the user the cost before running a long or high-resolution generation.

The older `agnes-video-v2.0` documents `height`, `width`, `num_frames`, and `frame_rate` instead of `size` and `seconds`. Do not mix the two parameter sets.

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

Error bodies observed in testing look like `{"code": "insufficient_user_quota", "message": "...", "data": null}`. Video create returned HTTP 403 with `insufficient_user_quota` when the account balance was $0, so a 403 can also mean no credit, not only a policy block.

## Not yet documented

The video `videos` object fields, aspect ratios other than 16:9, image edit input shape, and `MODEL_CATALOG.md`. Official docs: https://agnes-ai.com/doc/overview. Do not guess these; check the docs or ask the user.
