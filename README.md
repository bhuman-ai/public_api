# BHuman AI Studio API Docs

Last updated: June 2026.

This repository is the public GitHub endpoint reference for the BHuman AI Studio
API. The richer documentation lives on the BHuman website:

- https://www.bhuman.ai/docs
- https://www.bhuman.ai/docs/api

The generated Swagger UI and OpenAPI schema are available here:

- https://studio.bhuman.ai/swagger-ui/
- https://studio.bhuman.ai/swagger-ui/openapi.json

The live schema is `BHuman AI Studio API` version `0.2.0`.

## Current product model

Older BHuman help articles and videos may mention "Easy Mode" and "Advanced
Mode." Those terms describe an older product flow and should not be used for new
setup.

The current model is:

1. Create or import an AI Studio video template.
2. Define the variables that should change for each recipient.
3. Generate personalized videos through an AI Studio campaign, integration, or
   API request.
4. Use callback URLs or polling endpoints to collect share links, MP4 files,
   thumbnails, GIF previews, status, and failure details.

Personalized videos can start from either source:

1. **Recorded or uploaded base video in AI Studio**
   - Record or upload the presenter video.
   - Mark variables such as `first_name`, `company`, `product_name`, or
     `cta_url`.
   - Trigger personalized renders through a campaign or API request.

2. **Generated presenter video from Speakeasy**
   - Generate the presenter and voice in Speakeasy.
   - Import the finished project into AI Studio.
   - Mark variables in AI Studio.
   - Trigger personalized renders through campaign or API calls.

## Authentication

BHuman API keys use Basic Authentication.

Create an API key in BHuman:

1. Sign in to BHuman.
2. Open **Settings**.
3. Open **API keys**.
4. Generate a key.
5. Store the key ID and secret securely.

Example header:

```bash
Authorization: Basic BASE64_ENCODED_KEY_ID_AND_SECRET
```

Example cURL setup:

```bash
export BHUMAN_API_KEY_ID="your_api_key_id"
export BHUMAN_API_KEY_SECRET="your_api_key_secret"

export BHUMAN_BASIC_AUTH=$(printf "%s:%s" "$BHUMAN_API_KEY_ID" "$BHUMAN_API_KEY_SECRET" | base64)
```

Python helper:

```python
import base64

api_key_id = "your_api_key_id"
api_key_secret = "your_api_key_secret"

token = base64.b64encode(f"{api_key_id}:{api_key_secret}".encode()).decode()
headers = {"Authorization": f"Basic {token}"}
```

## Endpoint summary

All AI Studio API endpoints below use this host:

```text
https://studio.bhuman.ai
```

| Method | Endpoint | Purpose |
| :-- | :-- | :-- |
| `POST` | `/api/ai_studio/pipeline/campaign` | Generate campaign videos from ordered variables and recipient rows |
| `POST` | `/api/ai_studio/pipeline/zapier` | Generate videos from the structured automation payload shape |
| `POST` | `/api/ai_studio/pipeline/pabbly` | Generate videos from a simple object-style webhook payload |
| `POST` | `/api/ai_studio/pipeline/leadr` | Generate videos from Leadr pipeline data |
| `POST` | `/api/ai_studio/try_sample` | Generate videos directly from a video instance ID |
| `GET` | `/api/ai_studio/video_instances` | List AI Studio video instances |
| `GET` | `/api/ai_studio/video_instance?id={video_instance_id}` | Fetch one video instance |
| `GET` | `/api/ai_studio/campaigns` | List campaigns |
| `GET` | `/api/ai_studio/campaign?id={campaign_id}` | Fetch one campaign |
| `GET` | `/api/ai_studio/generated_video_by_video_instance_id?video_instance_id={video_instance_id}` | List generated videos for an instance |
| `GET` | `/api/ai_studio/generated_video_by_campaign_id?campaign_id={campaign_id}` | List generated videos for a campaign |
| `GET` | `/api/ai_studio/generated_video_by_id?id={generation_id}` | Fetch one generated video |
| `GET` | `/api/ai_studio/speakeasy_projects` | List Speakeasy projects available for import |
| `POST` | `/api/ai_studio/speakeasy_projects/import` | Import a Speakeasy project into AI Studio |
| `GET` | `/api/ai_studio/webhook` | List saved campaign webhooks |
| `POST` | `/api/ai_studio/webhook` | Create a saved campaign webhook |
| `POST` | `/api/ai_studio/videos_preview` | Create a video preview request |

For exact optional fields and additional operations, use Swagger:

- https://studio.bhuman.ai/swagger-ui/

## Generate videos from a campaign

Use this endpoint when your workflow is based on an AI Studio campaign.

```http
POST https://studio.bhuman.ai/api/ai_studio/pipeline/campaign
```

Request body:

| Field | Type | Required | Notes |
| :-- | :-- | :-- | :-- |
| `campaign_id` | `String` | Yes | Campaign ID |
| `variables` | `String[]` | Yes | Variable names in the same order as each row in `names` |
| `names` | `String[][]` | Yes | Recipient rows. Each row maps to `variables` by position |
| `callback_url` | `String` | No | BHuman sends completion details for generated videos |
| `assets` | `String[][]` | No | Asset URL rows used by dynamic backgrounds |
| `backgrounds` | `Background[]` | No | Dynamic background segments |
| `enable_lipsync` | `Boolean` | No | Enable lip sync when supported by the template |
| `enable_subtitles` | `Boolean` | No | Request subtitles when supported by the template |

Example:

```bash
curl -X POST "https://studio.bhuman.ai/api/ai_studio/pipeline/campaign" \
  -H "Authorization: Basic $BHUMAN_BASIC_AUTH" \
  -H "Content-Type: application/json" \
  -d '{
    "campaign_id": "YOUR_CAMPAIGN_ID",
    "variables": ["first_name", "company", "cta_url"],
    "names": [
      ["Alex", "ExampleCo", "https://example.com/start"],
      ["Jordan", "Sample Labs", "https://example.com/book"]
    ],
    "callback_url": "https://yourapp.com/bhuman/callback"
  }'
```

Accepted response:

```json
{
  "code": 200,
  "result": [
    "c10ee155-6202-4cec-9a40-dde536e2ab4e"
  ]
}
```

## Pabbly-style payload

Use the Pabbly endpoint when a webhook builder is easier with object-style
variables instead of ordered row arrays.

```http
POST https://studio.bhuman.ai/api/ai_studio/pipeline/pabbly
```

Example:

```json
{
  "campaign_id": "YOUR_CAMPAIGN_ID",
  "variables": {
    "first_name": "Alex",
    "company": "ExampleCo",
    "cta_url": "https://example.com/start"
  },
  "callback_url": "https://yourapp.com/bhuman/callback"
}
```

## Generate videos from a video instance

Use this endpoint when your workflow targets a specific AI Studio video
instance instead of a campaign.

```http
POST https://studio.bhuman.ai/api/ai_studio/try_sample
```

Example:

```bash
curl -X POST "https://studio.bhuman.ai/api/ai_studio/try_sample" \
  -H "Authorization: Basic $BHUMAN_BASIC_AUTH" \
  -H "Content-Type: application/json" \
  -d '{
    "video_instance_id": "YOUR_VIDEO_INSTANCE_ID",
    "variables": ["first_name", "company"],
    "names": [
      ["Alex", "ExampleCo"],
      ["Jordan", "Sample Labs"]
    ],
    "callback_url": "https://yourapp.com/bhuman/callback"
  }'
```

## Variables and recipient rows

`variables` defines the column order. `names` supplies the recipient values.

Example:

```json
{
  "variables": ["first_name", "company", "cta_url"],
  "names": [
    ["Alex", "ExampleCo", "https://example.com/start"],
    ["Jordan", "Sample Labs", "https://example.com/book"]
  ]
}
```

Common variables:

- `first_name`
- `company`
- `product_name`
- `plan_name`
- `cta_url`
- `booking_url`
- `renewal_date`
- `asset_url`

## Dynamic backgrounds

Dynamic backgrounds let a generated video show a website, image, video,
LinkedIn page, product page, landing page, or other URL behind or around the
presenter.

`assets` is a row-by-row matrix. Each recipient row can provide one or more URLs
used by the matching background segment.

Example:

```json
{
  "variables": ["first_name", "company"],
  "names": [
    ["Alex", "ExampleCo"],
    ["Jordan", "Sample Labs"]
  ],
  "assets": [
    ["https://example.com/product", "https://example.com/alex-demo.mp4"],
    ["https://example.com/pricing", "https://example.com/jordan-demo.mp4"]
  ],
  "backgrounds": [
    {
      "name": "product_page",
      "start": 0,
      "end": 3,
      "kind": "site",
      "config": {
        "mode": "circle",
        "position": "bottom-right",
        "scale": 0.4
      }
    },
    {
      "name": "demo_video",
      "start": 3,
      "end": -1,
      "kind": "link",
      "config": {
        "mode": "chroma",
        "color": "00FF00"
      }
    }
  ]
}
```

### Background

| Field | Type | Required | Notes |
| :-- | :-- | :-- | :-- |
| `name` | `String` | No | Reference name for debugging |
| `start` | `Number` | Yes | Segment start time in seconds |
| `end` | `Number` | Yes | Segment end time in seconds. Use `-1` for the rest of the video |
| `kind` | `String` | Yes | Common values include `link`, `site`, and `linkedin` |
| `config` | `BackgroundConfig` | Yes | Display settings |

### BackgroundConfig

| Field | Type | Notes |
| :-- | :-- | :-- |
| `mode` | `String` | Background display mode, such as `circle` or `chroma` |
| `scale` | `Number` | Scale factor for the presenter overlay |
| `position` | `String` | Overlay position, such as `center`, `bottom-right`, or `top-right` |
| `audio` | `Boolean` | Whether to use audio from the background asset |
| `brightness` | `Number` | Brightness for the non-covered background area |
| `color` | `String` | Chroma key color in HEX when using `chroma` |
| `similarity` | `Number` | Similarity level when using `chroma` |
| `blend` | `Number` | Blend level when using `chroma` |

## Completion callbacks and polling

If `callback_url` is provided, BHuman can send completion details for generated
videos. Store the generation ID so callbacks and polling responses can be
reconciled.

Completion fields:

| Field | Type | Notes |
| :-- | :-- | :-- |
| `id` | `String` | Generation ID |
| `status` | `String` | Processing, succeeded, failed, or related render state |
| `share_url` | `String` | Hosted video page or share destination when available |
| `url` | `String` | Downloadable MP4 URL |
| `thumbnail` | `String` | Generated thumbnail image URL |
| `gif` | `String` | Generated GIF preview URL |
| `message` | `String` | Failure or processing detail when available |
| `execution_name` | `String` | Pipeline execution identifier when available |
| `row_index` | `Int` | Recipient row index when available |

Example:

```json
{
  "id": "c10ee155-6202-4cec-9a40-dde536e2ab4e",
  "status": "succeeded",
  "share_url": "https://videos.bhuman.ai/video/...",
  "url": "https://assets.bhuman.ai/generated-video.mp4",
  "thumbnail": "https://assets.bhuman.ai/thumb.jpg",
  "gif": "https://assets.bhuman.ai/preview.gif"
}
```

If you do not use callbacks, poll generated video endpoints:

```http
GET https://studio.bhuman.ai/api/ai_studio/generated_video_by_video_instance_id?video_instance_id={video_instance_id}&page=0&size=100
GET https://studio.bhuman.ai/api/ai_studio/generated_video_by_campaign_id?campaign_id={campaign_id}&page=0&size=100
GET https://studio.bhuman.ai/api/ai_studio/generated_video_by_id?id={generation_id}
```

## Speakeasy imports

Use Speakeasy to generate the presenter and voice, then import that project into
AI Studio before marking variables or triggering campaign/API rendering.

List importable projects:

```http
GET https://studio.bhuman.ai/api/ai_studio/speakeasy_projects
```

Import a project:

```bash
curl -X POST "https://studio.bhuman.ai/api/ai_studio/speakeasy_projects/import" \
  -H "Authorization: Basic $BHUMAN_BASIC_AUTH" \
  -H "Content-Type: application/json" \
  -d '{
    "project_id": "SPEAKEASY_PROJECT_ID",
    "workspace_id": "OPTIONAL_WORKSPACE_ID"
  }'
```

## Saved webhooks

Create a reusable webhook destination:

```http
POST https://studio.bhuman.ai/api/ai_studio/webhook
```

Request body:

```json
{
  "name": "Production callback",
  "webhook": "https://yourapp.com/bhuman/callback"
}
```

List saved webhooks:

```http
GET https://studio.bhuman.ai/api/ai_studio/webhook
```

## Production checklist

- Use HTTPS callback URLs that can accept repeated delivery attempts.
- Store generation IDs so callbacks and polling responses can be reconciled.
- Keep variables stable after a campaign is wired into a production workflow.
- Send row-specific assets in the same order as recipient rows.
- Treat generated media URLs as outputs from an async render job, not as
  immediate inline responses.
- Check Swagger before launching custom integrations that rely on optional
  fields.

## Support

For API support, contact help@bhuman.ai.

Helpful links:

- API docs: https://www.bhuman.ai/docs/api
- Swagger UI: https://studio.bhuman.ai/swagger-ui/
