---
name: Generate a video and receive it via webhook
description: Submit a text- or image-to-video generation and receive the result through a webhook callback.
api: openapi/higgsfield-openapi-original.json
operations:
  - submit_generation   # POST /{model_id}
  - get_request_status  # GET /requests/{request_id}/status
generated: '2026-07-19'
method: generated
source: https://docs.higgsfield.ai/docs/guides/video + /how-to/webhooks
---

# Generate a video and receive it via webhook

Generate video with a Higgsfield video model (e.g. `sora-2/text-to-video`,
`veo3.1`, `kling-video/v2.5-turbo/pro/text-to-video`) and get notified on
completion instead of polling.

## Auth
`Authorization: Key {api_key}:{api_key_secret}` on every call. See
`authentication/higgsfield-authentication.yml`.

## Steps

1. **Submit with a webhook** — add the `hf_webhook` query parameter pointing at
   your endpoint:
   ```
   curl -X POST 'https://platform.higgsfield.ai/sora-2/text-to-video?hf_webhook=https://your.app/hooks/hf' \
     -H 'Authorization: Key {api_key}:{api_key_secret}' \
     -H 'Content-Type: application/json' \
     -d '{ "prompt": "A neon city street in the rain", "aspect_ratio": "16:9" }'
   ```
   Response: `{ status: "queued", request_id, status_url, cancel_url }`.

2. **Receive the callback** — Higgsfield POSTs to your webhook URL when the
   request reaches a final status. On `completed` the payload carries
   `video.url`. See `asyncapi/higgsfield-webhooks.yml` for the event shapes.

3. **(Optional) Reconcile by polling** — if a callback is missed, fall back to
   `GET /requests/{request_id}/status`.

4. **(Optional) Cancel** — `POST /requests/{request_id}/cancel` while still queued.

## Rules
- Final statuses: `completed`, `failed`, `nsfw`. Failed/NSFW are not charged.
- Store `request_id` to reconcile later; output files expire after ~7 days.
- Video timeouts are model-specific; a timeout marks the request failed (not charged).
