---
name: agnes-api
description: Call the Agnes AI API (OpenAI-compatible) for text, image, video, and multimodal generation. Use when the user wants to integrate with, call, or debug requests to Agnes AI, apihub.agnes-ai.com, or an `agnes-*` model.
---

Agnes AI exposes an OpenAI-compatible API. Existing OpenAI client code usually only needs three changes: base URL, API key, model name.

## Connection

- Base URL: `https://apihub.agnes-ai.com/v1`
- Auth header: `Authorization: Bearer $AGNES_API_KEY`
- Read the key from the environment. Never hardcode it, commit it, put it in front-end code, or echo it in output. If it leaks, tell the user to reset it in the Agnes console.

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

## What is confirmed vs not

Confirmed from the Agnes docs: the base URL, Bearer auth, the `/chat/completions` path, and the model `agnes-2.5-flash`.

Not yet documented here: image, video, and audio-video endpoint paths, other model names, parameters, response shapes, and error codes. Do not guess them. Ask the user for the model-specific integration page, or check the Agnes docs index (`/llms.txt` on their docs site), before writing code against those.

## Checklist when a request fails

1. Path is exactly `/v1/chat/completions` (no doubled `/v1`).
2. `Authorization: Bearer` header is present and the key is valid.
3. `model` matches a name the account can use.
