# Kling 4.0 API production notes

## Confirmed contract

- Endpoint: `POST /api/generate/submit`
- Model ID: `kling-4.0`
- Authentication: `Authorization: Bearer <POYO_API_KEY>`
- Status: `GET /api/generate/status/{task_id}`

## Input limits

| Field | Requirement | Constraint |
| --- | --- | --- |
| prompt | one of prompt / multi_prompt | Up to 26,000 characters; at least one non-whitespace character. Required for editing. |
| multi_prompt | one of prompt / multi_prompt | Shots with prompt (up to 512 characters) and duration (1–30 s, default 5); total 3–30 s. Not with `video_url`. |
| duration | optional | Integer 3–30. Default 5; multi-shot uses the shot total; editing keeps the source duration. |
| resolution | optional | 720p or 1080p; default 720p. |
| aspect_ratio | optional | auto, 16:9, 9:16, 1:1, or 21:9; text and reference modes only. |
| image_urls | optional | 1–2 frames (start, optional end) in image mode; at most 1 starting frame in reference mode; not in editing. |
| reference_image_urls | optional | Up to 10; with a starting frame, at most 10 images in total. |
| reference_video_urls | optional | Up to 5, 3–30 s each; not in editing. |
| video_url | optional | Source video for editing, 3–30 s. |
| kling_elements | optional | Up to 7 subjects: 2–4 images each (frontal first) or exactly one 3–30 s video; at most 3 video subjects; video subjects not in editing. |

Starting frames, reference images, reference videos, and subjects together cannot exceed 15 materials. Reference videos and video subjects together can last at most 30 seconds. All media URLs must be public HTTP or HTTPS URLs.

## Operational checklist

- Keep API keys in a server-side secret manager.
- Validate user input before calling the API.
- Persist `data.task_id` immediately after submission.
- Use five-second polling only for development; prefer `callback_url` in production.
- Make webhook processing idempotent and reconcile status before fulfilling an order.
- Copy output files to durable storage after the task finishes.
- Recheck the live model page before hard-coding prices.

## Failure handling

Treat non-2xx HTTP responses and non-success API `code` values as errors. Preserve the HTTP status, API code, and a redacted response body for debugging.

Schema: [PoYo API documentation](https://docs.poyo.ai/api-manual/video-series/kling-4-0?utm_source=github&utm_medium=readme&utm_campaign=kling-4.0-api)
