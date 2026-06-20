# BHuman API Docs

Last updated: June 2026.

This repository is the public GitHub endpoint reference for the BHuman API. The
canonical, latest docs live on the BHuman website:

- https://www.bhuman.ai/docs/api

The generated Swagger UI and OpenAPI schema are available here:

- https://studio.bhuman.ai/swagger-ui/
- https://studio.bhuman.ai/swagger-ui/openapi.json

Use the BHuman API with AI Studio templates, campaigns, generated videos, and
store products.

## Important note about older docs

Older help center articles, screenshots, and videos may mention "Easy Mode" and
"Advanced Mode." Those terms describe an older product flow and should not be
used for new setup.

The current model is:

1. Create or import an AI Studio video template.
2. Define the variables that should change per recipient.
3. Generate personalized videos through AI Studio, a campaign, or the API.
4. Use callback URLs or polling endpoints to collect video, thumbnail, and GIF
   assets.

The `BackgroundConfig.mode` field shown later in this guide is still valid, but
it only controls how a dynamic background is displayed (`basic`, `circle`, or
`chroma`). It is unrelated to the old Easy Mode or Advanced Mode flow.

## Common setup paths

Personalized videos can start from either source:

1. **Recorded or uploaded base video in AI Studio**
   - Record or upload a presenter video.
   - Mark variables such as `first_name`, `company`, `product_name`, or `cta_url`.
   - Use the API to pass recipient-specific values and optional assets.

2. **Generated presenter video from Speakeasy**
   - Generate the presenter and voice in Speakeasy.
   - Import the result into AI Studio.
   - Mark variables in AI Studio.
   - Trigger personalized renders through campaign or template API calls.

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

AUTH_HEADER=$(printf "%s:%s" "$BHUMAN_API_KEY_ID" "$BHUMAN_API_KEY_SECRET" | base64)
```

Python helper:

```python
import base64

api_key_id = "your_api_key_id"
api_key_secret = "your_api_key_secret"

token = base64.b64encode(f"{api_key_id}:{api_key_secret}".encode()).decode()
headers = {"Authorization": f"Basic {token}"}
```

## IDs and URLs

Most API calls require an AI Studio video instance ID, a campaign ID, or a
generation ID.

Examples:

- Campaign URL: `https://app.bhuman.ai/campaign/generatedVideos/b3d2bfa8-fa75-49ed-9f8b-bb13452f49fd`
- Instance URL: `https://app.bhuman.ai/instance/7d859e18-dc89-446f-9718-9533bb178a75`

The final path segment is the ID:

- Campaign ID: `b3d2bfa8-fa75-49ed-9f8b-bb13452f49fd`
- Video instance ID: `7d859e18-dc89-446f-9718-9533bb178a75`

## Endpoint summary

| Method | Endpoint | Purpose |
| :-- | :-- | :-- |
| `GET` | `https://studio.bhuman.ai/api/ai_studio/video_instances` | List AI Studio video instances |
| `GET` | `https://studio.bhuman.ai/api/ai_studio/video_instance` | Get one video instance |
| `POST` | `https://studio.bhuman.ai/api/ai_studio/try_sample` | Generate videos from a video instance |
| `POST` | `https://studio.bhuman.ai/api/ai_studio/pipeline/campaign` | Generate videos from a campaign |
| `GET` | `https://studio.bhuman.ai/api/ai_studio/generated_video_by_video_instance_id` | List generated videos for an instance |
| `GET` | `https://studio.bhuman.ai/api/ai_studio/generated_video_by_campaign_id` | List generated videos for a campaign |
| `GET` | `https://studio.bhuman.ai/api/ai_studio/generated_video_by_id` | Get one generated video |
| `GET` | `https://user.bhuman.ai/api/workspace` | Get workspace data |
| `GET` | `https://store.bhuman.ai/api/store/product` | List store products |
| `POST` | `https://store.bhuman.ai/api/store/product/use` | Use a store product |
| `GET` | `https://store.bhuman.ai/api/store/settings` | Get store categories and tags |

## Generate videos from a campaign

Use this endpoint when your workflow is based on a campaign.

```http
POST https://studio.bhuman.ai/api/ai_studio/pipeline/campaign
```

Request body:

| Field | Type | Required | Notes |
| :-- | :-- | :-- | :-- |
| `campaign_id` | `Uuid` | Yes | Campaign ID |
| `variables` | `String[]` | Yes | Variable names in the same order as each row in `names` |
| `names` | `String[][]` | Yes | Recipient rows. Each row maps to `variables` by position |
| `callback_url` | `String` | No | BHuman sends one callback per generated video |
| `backgrounds` | `Background[]` | No | Dynamic background segments |
| `assets` | `String[][]` | No | Asset URL rows used by dynamic backgrounds |

Example:

```bash
curl -X POST "https://studio.bhuman.ai/api/ai_studio/pipeline/campaign" \
  -H "Authorization: Basic $AUTH_HEADER" \
  -H "Content-Type: application/json" \
  -d '{
    "campaign_id": "d744af70-cd3c-4a41-8bfe-89238f937a3b",
    "variables": ["first_name", "company"],
    "names": [
      ["Alex", "ExampleCo"],
      ["Jordan", "Sample Labs"]
    ],
    "callback_url": "https://example.com/webhooks/bhuman"
  }'
```

Immediate response:

```json
{
  "result": [
    "c10ee155-6202-4cec-9a40-dde536e2ab4e"
  ]
}
```

If `callback_url` is provided, BHuman sends a `POST` callback for each
generation.

Callback payload:

| Field | Type | Notes |
| :-- | :-- | :-- |
| `id` | `Uuid` | Generation ID |
| `status` | `String` | `succeeded`, `failed`, or `processing` |
| `video_url` | `String` | Hosted playback URL |
| `url` | `String` | Downloadable MP4 URL |
| `thumbnail` | `String` | Downloadable thumbnail URL |
| `gif` | `String` | Downloadable GIF URL |

## Generate videos from a video instance

Use this endpoint when your workflow targets a specific AI Studio video
instance.

```http
POST https://studio.bhuman.ai/api/ai_studio/try_sample
```

Request body:

| Field | Type | Required | Notes |
| :-- | :-- | :-- | :-- |
| `video_instance_id` | `Uuid` | Yes | AI Studio video instance ID |
| `variables` | `String[]` | Yes | Variable names in the same order as each row in `names` |
| `names` | `String[][]` | Yes | Recipient rows. Each row maps to `variables` by position |
| `callback_url` | `String` | No | BHuman sends one callback per generated video |
| `backgrounds` | `Background[]` | No | Dynamic background segments |
| `assets` | `String[][]` | No | Asset URL rows used by dynamic backgrounds |

Example:

```bash
curl -X POST "https://studio.bhuman.ai/api/ai_studio/try_sample" \
  -H "Authorization: Basic $AUTH_HEADER" \
  -H "Content-Type: application/json" \
  -d '{
    "video_instance_id": "7d859e18-dc89-446f-9718-9533bb178a75",
    "variables": ["first_name", "company"],
    "names": [
      ["Alex", "ExampleCo"],
      ["Jordan", "Sample Labs"]
    ],
    "callback_url": "https://example.com/webhooks/bhuman"
  }'
```

Immediate response:

```json
{
  "result": [
    "c10ee155-6202-4cec-9a40-dde536e2ab4e"
  ]
}
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

For a generic customer workflow, common variables are:

- `first_name`
- `company`
- `product_name`
- `plan_name`
- `cta_url`
- `booking_url`
- `renewal_date`

## Dynamic backgrounds

Dynamic backgrounds let a generated video show a website, image, video, LinkedIn
page, product page, landing page, or other URL behind or around the presenter.

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
      "name": "website",
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
      "name": "demo-video",
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
| `kind` | `String` | Yes | One of `link`, `site`, or `linkedin` |
| `config` | `BackgroundConfig` | Yes | Display settings |

### BackgroundConfig

| Field | Type | Default | Notes |
| :-- | :-- | :-- | :-- |
| `mode` | `String` | Required | Background display mode: `basic`, `circle`, or `chroma` |
| `scale` | `Number` | `0.4` | Scale factor for the presenter overlay |
| `position` | `String` | `bottom-right` | Overlay position, such as `center`, `bottom`, `bottom-left`, or `top-right` |
| `audio` | `Boolean` | `false` | Whether to use audio from the background asset |
| `brightness` | `Number` | `1` | Brightness for the non-covered background area |
| `color` | `String` | `00FF00` | Chroma key color in HEX when using `chroma` |
| `similarity` | `Number` | `0.1` | Similarity level when using `chroma` |
| `blend` | `Number` | `0.1` | Blend level when using `chroma` |

## Poll generated videos

If you do not provide `callback_url`, poll the generated video endpoints.

### By video instance ID

```http
GET https://studio.bhuman.ai/api/ai_studio/generated_video_by_video_instance_id?video_instance_id={video_instance_id}&page=0&size=100
```

### By campaign ID

```http
GET https://studio.bhuman.ai/api/ai_studio/generated_video_by_campaign_id?campaign_id={campaign_id}&page=0&size=100
```

### By generation ID

```http
GET https://studio.bhuman.ai/api/ai_studio/generated_video_by_id?id={generation_id}
```

Generated video response fields:

| Field | Type | Notes |
| :-- | :-- | :-- |
| `id` | `Uuid` | Generation ID |
| `user_id` | `String` | User ID |
| `video_id` | `Uuid` | Video ID |
| `actor_id` | `Uuid` | Actor ID |
| `video_instance_id` | `Uuid` | Present when generated from an instance |
| `campaign_id` | `Uuid` | Present when generated from a campaign |
| `url` | `String` | Downloadable MP4 URL |
| `vimeo_url` | `String` | Hosted playback URL |
| `thumbnail` | `String` | Thumbnail URL |
| `gif` | `String` | GIF URL |
| `status` | `String` | Generation status |
| `execution_name` | `String` | Execution ID to share with support if generation fails |
| `row_index` | `Int` | Row index from the submitted `names` payload |
| `text` | `String[]` | Submitted variable values for that row |
| `message` | `String` | Failure or status message when available |
| `created_at` | `Datetime` | Creation timestamp |
| `updated_at` | `Datetime` | Update timestamp |

## Video instances

### List video instances

```http
GET https://studio.bhuman.ai/api/ai_studio/video_instances?page=0&size=100
```

Parameters:

| Field | Type | Required | Notes |
| :-- | :-- | :-- | :-- |
| `page` | `Int` | Yes | Starts at `0` |
| `size` | `Int` | Yes | Maximum `100` |

### Get one video instance

```http
GET https://studio.bhuman.ai/api/ai_studio/video_instance?id={video_instance_id}
```

## Workspace

```http
GET https://user.bhuman.ai/api/workspace
```

Optional query parameter:

| Field | Type | Required | Notes |
| :-- | :-- | :-- | :-- |
| `id` | `Uuid` | No | Workspace ID |

## Store products

### List products

```http
GET https://store.bhuman.ai/api/store/product
```

### Use a product

```http
POST https://store.bhuman.ai/api/store/product/use
```

Request body:

| Field | Type | Required | Notes |
| :-- | :-- | :-- | :-- |
| `id` | `Uuid` | Yes | Product ID |
| `workspace_id` | `Uuid` | Yes | Workspace ID |
| `folder_id` | `Uuid` | No | Existing folder ID |
| `video_instance_id` | `Uuid` | No | Existing video instance ID |

If `folder_id` and `video_instance_id` are omitted, BHuman creates a new folder
and video instance.

If `video_instance_id` is provided, `folder_id` must also be provided.

### Categories and tags

```http
GET https://store.bhuman.ai/api/store/settings
```

## PHP SDK

Community PHP SDK:

- https://github.com/henryreith/BHuman-PHP-SDK

## Support

For API support, contact help@bhuman.ai.

Helpful links:

- API docs: https://www.bhuman.ai/docs/api
- Help center: https://help.bhuman.ai
