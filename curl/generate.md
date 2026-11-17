# Kling 4.0 API: cURL quickstart

```bash
export POYO_API_KEY="your-api-key"
export POYO_BASE_URL="https://api.poyo.ai"
```

## Text-to-video

```bash
curl -X POST "$POYO_BASE_URL/api/generate/submit" \
  -H "Authorization: Bearer $POYO_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
  "model": "kling-4.0",
  "input": {
    "prompt": "A street food vendor at a night market flips noodles in a flaming wok, sparks and steam rising under warm string lights. Slow push-in from a medium shot to a close-up of the wok. Sizzling oil, crowd chatter, and the vendor calling out, \"Two more coming up!\"",
    "duration": 5,
    "resolution": "720p",
    "aspect_ratio": "16:9"
  }
}'
```

## First frame (image-to-video)

```bash
curl -X POST "$POYO_BASE_URL/api/generate/submit" \
  -H "Authorization: Bearer $POYO_API_KEY" \
  -H "Content-Type: application/json" \
  -d @payloads/kling-4.0-image-to-video.json
```

Add a second URL to `image_urls` to set the last frame as well. Omit `aspect_ratio` in image mode.

## Reference image

```bash
curl -X POST "$POYO_BASE_URL/api/generate/submit" \
  -H "Authorization: Bearer $POYO_API_KEY" \
  -H "Content-Type: application/json" \
  -d @payloads/kling-4.0-reference-image.json
```

Refer to inputs by array order: `@Image1`, `@Video1`, `@Element1`.

## Multi-shot

```bash
curl -X POST "$POYO_BASE_URL/api/generate/submit" \
  -H "Authorization: Bearer $POYO_API_KEY" \
  -H "Content-Type: application/json" \
  -d @payloads/kling-4.0-multi-shot.json
```

Send `multi_prompt` instead of `prompt`. A top-level `duration` must equal the shot total (here 4 + 6 = 10).

## Video editing

```bash
curl -X POST "$POYO_BASE_URL/api/generate/submit" \
  -H "Authorization: Bearer $POYO_API_KEY" \
  -H "Content-Type: application/json" \
  -d @payloads/kling-4.0-video-edit.json
```

`video_url` selects editing. Omit `duration` to keep the source length.

## Check status

Save `data.task_id` from the submit response, then poll:

```bash
curl "$POYO_BASE_URL/api/generate/status/<task_id>" \
  -H "Authorization: Bearer $POYO_API_KEY"
```

For production, add a public HTTPS `callback_url` to the top-level request and make the receiver idempotent.

See the [full README](../README.md) and [production notes](../docs/production-notes.md).
