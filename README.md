# Kling 4.0 API: cURL, Python, and Node.js Examples

[![Kling 4.0 on PoYo](https://img.shields.io/badge/Try%20on%20PoYo-kling--4.0-84cc16)](https://poyo.ai/models/kling-4-0?utm_source=github&utm_medium=readme&utm_campaign=kling-4.0-api) [![API docs](https://img.shields.io/badge/API-docs-22d3ee)](https://docs.poyo.ai/api-manual/video-series/kling-4-0?utm_source=github&utm_medium=readme&utm_campaign=kling-4.0-api) [![Check](https://github.com/PoyoAPI/Kling-4.0-API/actions/workflows/check.yml/badge.svg)](https://github.com/PoyoAPI/Kling-4.0-API/actions/workflows/check.yml)

Runnable examples for Kuaishou **Kling 4.0** through PoYo's async video API: 3–30 second videos at 720p or 1080p from text, first and last frames, multi-shot prompts, or image, video, and subject references, plus prompt-based editing of an existing video.

**Answer:** To use the Kling 4.0 API, send a server-side request to `POST https://api.poyo.ai/api/generate/submit` with model `kling-4.0` and a PoYo API key, then poll `GET https://api.poyo.ai/api/generate/status/{task_id}`. The examples in this repository run with Python 3.9+, Node.js 22+, or cURL.

Last verified against the live PoYo schema and pricing on **November 19, 2026**.

## Kling 4.0 at a glance

- **Model ID:** `kling-4.0` on `POST https://api.poyo.ai/api/generate/submit`
- **Output:** 720p (default) or 1080p, 3–30 seconds; ordinary generation defaults to 5 seconds
- **Modes:** text-to-video, image-to-video (first frame, optional last frame), reference-to-video, multi-shot, and video editing
- **References:** up to 10 images, 5 videos, and 7 subjects, at most 15 materials in total
- **Aspect ratios:** auto, 16:9, 9:16, 1:1, 21:9 (text and reference modes)
- **Audio:** native stereo audio is generated with every video; there is no on/off field
- **Price:** from 28.5 credits ($0.1425) per second at 720p and 42.5 credits ($0.2125) per second at 1080p

## Quickstart

1. Create a key in the [PoYo dashboard](https://poyo.ai/dashboard/api-key?utm_source=github&utm_medium=readme&utm_campaign=kling-4.0-api).
2. Copy `.env.example` to `.env` and replace the placeholder.
3. Run one of the examples. Each submits the task and polls until `finished` or `failed`.

```dotenv
POYO_API_KEY=YOUR_POYO_API_KEY_HERE
POYO_BASE_URL=https://api.poyo.ai
```

### Python

```bash
python python/generate.py
python python/generate.py payloads/kling-4.0-multi-shot.json
```

[`python/generate.py`](python/generate.py) uses only the Python standard library.

### Node.js

```bash
node node/index.mjs
node node/index.mjs payloads/kling-4.0-image-to-video.json
```

### cURL

```bash
curl -X POST "https://api.poyo.ai/api/generate/submit" \
  -H "Authorization: Bearer $POYO_API_KEY" \
  -H "Content-Type: application/json" \
  -d @payloads/kling-4.0-text-to-video.json
```

Status polling and the other modes are in [curl/generate.md](curl/generate.md).

## Example requests

Every file in [`payloads/`](payloads) is a complete request body with public input media, so it runs as-is.

| File | Mode | Settings | Credits |
| --- | --- | --- | --- |
| [`kling-4.0-text-to-video.json`](payloads/kling-4.0-text-to-video.json) | Text-to-video | 720p, 5 s, 16:9 | 142.5 ($0.7125) |
| [`kling-4.0-image-to-video.json`](payloads/kling-4.0-image-to-video.json) | First frame | 720p, 5 s, no `aspect_ratio` | 142.5 ($0.7125) |
| [`kling-4.0-reference-image.json`](payloads/kling-4.0-reference-image.json) | `@Image1` reference | 720p, 5 s, 16:9 | 142.5 ($0.7125) |
| [`kling-4.0-multi-shot.json`](payloads/kling-4.0-multi-shot.json) | `multi_prompt`, 4 s + 6 s | 720p, 10 s total | 285 ($1.425) |
| [`kling-4.0-video-edit.json`](payloads/kling-4.0-video-edit.json) | Video editing, 10.05 s source | 720p, 10 s output | 357 ($1.785) |

The edit is billed as 10 output seconds plus max(ceil(10.05) = 11, 10) = 21 billed seconds at 17 credits.

## Input modes

The supplied media selects the mode automatically.

| Mode | Selected when |
| --- | --- |
| Video editing | `video_url` is provided |
| Reference-to-video | `reference_image_urls`, `reference_video_urls`, or `kling_elements` is non-empty |
| Image-to-video | `image_urls` is non-empty without reference materials |
| Text-to-video | No media is provided |

- **Image mode:** put the starting frame first in `image_urls` and an optional ending frame second, and omit `aspect_ratio`.
- **Reference mode:** `image_urls` may hold one optional starting frame. Refer to inputs as `@Image1`, `@Video1`, and `@Element1` in array order; the older `@image_1` and `@video_1` forms remain supported.
- **Editing:** send `video_url` and `prompt`. Editing accepts reference images and image-based subjects, but not `image_urls`, reference videos, video subjects, `multi_prompt`, or `aspect_ratio`. Omit `duration` to keep the source duration.
- **Multi-shot:** send `multi_prompt` instead of `prompt`. Each shot has a prompt of up to 512 characters and a duration of 1–30 seconds (default 5); shots must total 3–30 seconds, and a top-level `duration` must equal that total.

## Request fields

| Field | Requirement | Meaning |
| --- | --- | --- |
| prompt | one of `prompt` / `multi_prompt` | Video instructions, up to 26,000 characters. Required for editing. |
| multi_prompt | one of `prompt` / `multi_prompt` | Array of shots: `{ "prompt", "duration" }`. Generation modes only. |
| duration | optional | Integer from 3 to 30 seconds. Default 5; multi-shot uses the shot total; editing keeps the source duration. |
| resolution | optional | `720p` or `1080p`. Default `720p`. |
| aspect_ratio | optional | `auto`, `16:9`, `9:16`, `1:1`, or `21:9`, for text and reference modes. Without materials, defaults to 16:9. |
| image_urls | optional | Starting frame, plus an optional ending frame in image mode. |
| reference_image_urls | optional | Up to 10 images; with a starting frame, at most 10 images in total. |
| reference_video_urls | optional | Up to 5 videos, 3–30 seconds each. Not allowed in editing. |
| video_url | optional | Source video for editing, 3–30 seconds. |
| kling_elements | optional | Up to 7 subjects, each with 2–4 images (`element_input_urls`, frontal view first) or exactly one 3–30 second video (`element_input_video_urls`); at most 3 video subjects. |

Starting frames, reference images, reference videos, and subjects together cannot exceed 15 materials; the images inside one subject count as one. Reference videos and video subjects together can last at most 30 seconds.

## Pricing

1 credit = $0.005.

| Resolution | No video input | With video input |
| --- | --- | --- |
| 720p | 28.5 credits ($0.1425) per second | 17 credits ($0.085) per billed second |
| 1080p | 42.5 credits ($0.2125) per second | 25.5 credits ($0.1275) per billed second |

With a source, reference, or subject video, billed seconds = output duration + max(ceil(total input video duration), output duration). Confirm current prices on the [model page](https://poyo.ai/models/kling-4-0?utm_source=github&utm_medium=readme&utm_campaign=kling-4.0-api) before a production rollout.

### Cost per clip

Without video input:

| Clip length | 720p | 1080p |
| --- | --- | --- |
| 5 seconds | 142.5 credits ($0.7125) | 212.5 credits ($1.0625) |
| 10 seconds | 285 credits ($1.425) | 425 credits ($2.125) |
| 30 seconds | 855 credits ($4.275) | 1,275 credits ($6.375) |

## Writing Kling 4.0 prompts

- **Direct the camera:** name the move (push-in, orbit, tracking shot) and tie it to the action.
- **Write the sound:** put spoken lines in quotes and describe ambience and effects; audio is always generated.
- **Anchor references:** give every `@Image`, `@Video`, and `@Element` one job, for example "@Element1 walks through the square in @Image1".
- **Split long stories:** use `multi_prompt` with one beat per shot instead of one very long prompt.
- **Draft, then render:** iterate at 720p and switch `resolution` to `1080p` for the final cut.

## Async production flow

```text
submit -> persist data.task_id -> poll during development -> receive callback_url webhook in production
```

Treat `finished` and `failed` as terminal states. Accept a webhook only for task IDs your system created, process it idempotently, and reconcile the final state through the status endpoint. Copy output videos to your own storage after the task finishes.

## Common errors

Always read `code` in the response body as well as the HTTP status.

| `code` | Meaning and fix |
| --- | --- |
| 400 | A field is invalid, for example `duration` outside 3–30, both `prompt` and `multi_prompt`, or `aspect_ratio` in image or editing mode. Correct the request and resubmit. |
| 401 | Missing or invalid key: send `Authorization: Bearer <POYO_API_KEY>`. |
| 402 | Insufficient credits: top up in the [dashboard](https://poyo.ai/dashboard?utm_source=github&utm_medium=readme&utm_campaign=kling-4.0-api). |
| 429 | Back off and retry. |

A task that fails after submission returns `status: "failed"` with `error_message`. See [error codes](https://docs.poyo.ai/api-manual/error-codes?utm_source=github&utm_medium=readme&utm_campaign=kling-4.0-api).

## FAQ

**How long can a Kling 4.0 video be?** 3 to 30 seconds, in whole seconds, at 720p or 1080p.

**How much does Kling 4.0 cost per second?** 28.5 credits ($0.1425) at 720p and 42.5 credits ($0.2125) at 1080p without video input. With video input, the rates are 17 and 25.5 credits per billed second.

**Can I turn the audio off?** No. Kling 4.0 on PoYo returns native stereo audio with every video, and the request has no audio switch.

**Can Kling 4.0 edit an existing video?** Yes. Send `video_url` with a 3–30 second source video and a `prompt` that describes the change.

**How many references can one request use?** Up to 10 images (including an optional starting frame), 5 videos, and 7 subjects, with at most 15 materials in total.

**How is Kling 4.0 different from Kling 3.0?** Kling 4.0 adds 30-second clips, reference videos and subjects, and video editing. Kling 3.0 tops out at 15 seconds but offers native 4K. See [Kling-3.0-API](https://github.com/PoyoAPI/Kling-3.0-API).

## Repository map

- [`payloads/`](payloads) — complete request bodies for each mode
- [`python/generate.py`](python/generate.py) — standard-library Python example
- [`node/index.mjs`](node/index.mjs) — dependency-free Node.js example
- [`curl/generate.md`](curl/generate.md) — copy-paste cURL requests
- [`docs/production-notes.md`](docs/production-notes.md) — limits and rollout checklist

## Related repositories

- [Kling-Image-4.0-API](https://github.com/PoyoAPI/Kling-Image-4.0-API) — Kling Image 4.0: native 8K images, image sets, and long webpage or ecommerce page designs from the Kling 4.0 family
- [Kling-3.0-API](https://github.com/PoyoAPI/Kling-3.0-API) — Kling 3.0 with native 4K
- [Seedance-2.5-API](https://github.com/PoyoAPI/Seedance-2.5-API) — Seedance 2.5 multimodal-reference video
- [awesome-poyo-api](https://github.com/PoyoAPI/awesome-poyo-api) — every PoYo model repository in one list
- [poyo-devtools](https://github.com/PoyoAPI/poyo-devtools) — PoYo CLI, MCP bridge, and Agent Skill

## Links

- [Kling 4.0 API on PoYo](https://poyo.ai/models/kling-4-0?utm_source=github&utm_medium=readme&utm_campaign=kling-4.0-api)
- [Kling 4.0 API reference](https://docs.poyo.ai/api-manual/video-series/kling-4-0?utm_source=github&utm_medium=readme&utm_campaign=kling-4.0-api)
- [PoYo model catalog](https://poyo.ai/models?utm_source=github&utm_medium=readme&utm_campaign=kling-4.0-api)

## License

MIT
