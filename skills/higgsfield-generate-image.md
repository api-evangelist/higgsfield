---
name: Generate an image from a text prompt
description: Submit a text-to-image generation to the Higgsfield API and retrieve the result URL.
api: openapi/higgsfield-openapi-original.json
operations:
  - submit_generation   # POST /{model_id}
  - get_request_status  # GET /requests/{request_id}/status
generated: '2026-07-19'
method: generated
source: https://docs.higgsfield.ai/docs/guides/images
---

# Generate an image from a text prompt

Use the Higgsfield API to turn a text prompt into an image. The API is
asynchronous: you enqueue a request, then poll status (or use a webhook).

## Auth
Send your credentials on every call:
`Authorization: Key {api_key}:{api_key_secret}` (generate them at
https://cloud.higgsfield.ai/). See `authentication/higgsfield-authentication.yml`.

## Steps

1. **Submit the generation** — `POST https://platform.higgsfield.ai/{model_id}`
   with a flagship image model, e.g. `higgsfield-ai/soul/standard`:
   ```
   curl -X POST 'https://platform.higgsfield.ai/higgsfield-ai/soul/standard' \
     -H 'Authorization: Key {api_key}:{api_key_secret}' \
     -H 'Content-Type: application/json' \
     -d '{ "prompt": "A serene mountain landscape at sunset",
           "aspect_ratio": "16:9", "resolution": "720p" }'
   ```
   The response is `{ status: "queued", request_id, status_url, cancel_url }`.

2. **Poll status** — `GET /requests/{request_id}/status` until `status` is a
   final value (`completed`, `failed`, or `nsfw`).

3. **Read the result** — on `completed`, take the URL from `images[0].url`.

## Rules
- `failed` and `nsfw` requests are not charged (auto-refunded). See
  `errors/higgsfield-problem-types.yml`.
- Output files are retained a minimum of 7 days — download promptly.
- There is no idempotency key; a resubmit creates a new request.
- For production, prefer a webhook (`?hf_webhook=...`) over polling — see
  `asyncapi/higgsfield-webhooks.yml`.
