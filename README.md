# BHuman AI Studio REST API

Updated October 2, 2026. **Integration qualification is in progress.** Deployed
schemas and source were reviewed, and logged-out/invalid-credential checks pass.
Fresh ordinary-account generation, playable output, quota failures and retry/charge
reconciliation still require end-to-end qualification. This is not a launch
certification.

Use the public [API guide](https://www.bhuman.ai/docs/api) for customer setup.
The developer hub and hosted MCP guide are being prepared for publication.
Customer setup requires no private repository or local MCP server.

## Choose the correct integration

- **AI Studio REST:** personalize an existing template/campaign with recipient
  values. Prepare the source video and variables in the BHuman app first.
- **BHuman hosted MCP:** let Claude Code or Cursor create a Speakeasy presenter
  video; separate `studio_*` tools operate AI Studio workflows. The endpoint is
  `https://speakeasy.bhuman.ai/api/mcp`.

A Speakeasy project is not an AI Studio template. Finished Speakeasy footage can
be imported in the app and configured for Studio personalization.

## Prerequisites and costs

Create an account at [BHuman](https://app.bhuman.ai), use owned media and a campaign
you can access, and review **Settings → Plan & usage** before generating.
Account/product entitlements and credits apply to API requests. Free, Growth,
Scale and Ultimate are account plans, not an API quota bypass; legacy/promotional
balances can have product restrictions. Confirm restricted access with support.

Generation uses production credits. Review the exact campaign estimate; do not
assume every quality/mode costs the same. No separate free API sandbox, fixed
public per-minute REST limit or guaranteed rendering SLA is documented.

## REST authentication

In **Settings → API Keys → Generate new key**, obtain your **Client ID** and
**Client Secret**. These are HTTP Basic username/password credentials for the
integration generation routes below. Keep them in a private server environment or
secret manager. Do not expose credentials in browser code, logs or screenshots.

An **MCP Key** beginning `bhm_mcp_` is a separate bearer credential. Generate it
with **Generate MCP key**. MCP keys expire after 90 days; replacement revokes older
keys for the account. This expiry policy does not apply to REST credentials.

## Generate one personalized video

Create a ready campaign with one variable named `first_name`. Save this as
`campaign.json`, replacing the placeholder with your owned campaign UUID:

```json
{
  "campaign_id": "00000000-0000-0000-0000-000000000000",
  "generation_mode": "keep_original",
  "variables": ["first_name"],
  "names": [["Alex"]]
}
```

Set `BHUMAN_CLIENT_ID` and `BHUMAN_CLIENT_SECRET` privately, review the credit
charge, then submit **once**:

```bash
curl --fail-with-body --request POST \
  'https://studio.bhuman.ai/api/ai_studio/pipeline/campaign' \
  --user "$BHUMAN_CLIENT_ID:$BHUMAN_CLIENT_SECRET" \
  --header 'Content-Type: application/json' \
  --data @campaign.json
```

Acceptance returns `{"code":200,"result":["GENERATION_UUID"]}`. Save the IDs.
Rendering is asynchronous. Each `names` row must match the ordered `variables`
array; values replace only the variable, not its surrounding sentence. A batch
accepts at most 2,000 rows; start with one.

`generation_mode` accepts `keep_original` (the default when missing/blank) and
`full_script`. Send it explicitly. These are Studio modes, independent of
Speakeasy quality choices. Keep Original supports up to 100 personalized sections,
each at most 10 seconds. Optional `assets`/`backgrounds` must align with recipient
rows and background columns; omit them from the first request.

## Track and retrieve output

Poll an accepted individual ID about every 10 seconds with a bounded deadline:

```bash
curl --fail-with-body --get \
  'https://studio.bhuman.ai/api/ai_studio/generated_video_by_id' \
  --data-urlencode 'id=YOUR_RETURNED_GENERATION_UUID'
```

The envelope is `{code, result: generatedVideo}`. Wait for `result.status` to be
`succeeded` and `result.url` to contain the MP4. Open it and verify video/audio
playback. `share_url` is the hosted page; `thumbnail` and `gif` may be absent until
ready. `failed` is terminal; preserve the ID and public failure message.

**Access limitation:** the current individual-result route relies on possession
of the generation ID rather than account authentication. Treat IDs and media URLs
as private. Do not publish them in shared examples or logs.

## Endpoint access boundaries

| Endpoint | Integration behavior |
| --- | --- |
| `POST /api/ai_studio/pipeline/campaign` | Basic Authentication; generates from an owned campaign |
| `POST /api/ai_studio/try_sample` | Basic Authentication; use `video_instance_id`, ordered variables and names |
| `GET /api/ai_studio/generated_video_by_id?id=UUID` | Individual result by sensitive returned ID |
| Template/campaign lists and bulk result lists | First-party bearer-token routes; do not assume Basic Authentication works |

For example, `generated_video_by_campaign_id` requires a BHuman user bearer token,
`campaign_id`, `page` and `size`. Prepare campaigns in the app; use an individual
ID or callback for this Basic-auth integration.

[Swagger](https://studio.bhuman.ai/swagger-ui/) and
[live OpenAPI JSON](https://studio.bhuman.ai/swagger-ui/openapi.json) are public.
The service schema version is `0.2.0`. It includes internal/admin and completion
webhook ingress routes; inclusion is not a grant of customer access. Never call
provider completion, admin, credit adjustment or raw pipeline operations.

## Callbacks and retries

Optional `callback_url` points to your own HTTPS receiver. Completion callbacks
use `id`, `campaign_id`, `status`, `video_url` (hosted page), `url` (MP4),
`thumbnail`, `gif` and `campaign_result_id`. Failure can leave media strings empty.
Handle repeated deliveries once per ID/event and reconcile against stored IDs.
No callback-signature or delivery retry SLA is promised by this reference.

When the saved campaign enables WhatsApp output, the main completion and later
`event=whatsapp.ready` callback are separate; the later event supplies
`whatsapp_video_url`. This optional path needs account-specific qualification.

REST generation has no documented idempotency key. **Disable automatic paid POST
retries.** A lost response or timeout can mean accepted work. Check the campaign
in the app before resubmitting; repeat status GETs instead when IDs are known.

Missing/invalid credentials on a valid body return 401. Missing fields or malformed
UUIDs can return 422 before authentication. Valid JSON with invalid variable mapping
can return 400. Insufficient quota is a business failure; a render can fail after
initial HTTP 200. Read HTTP status and the `code/result/error` payload, preserve
references and review balance before a fresh start.

## MCP scope

The hosted MCP's deployed-source options are `quality: standard|hd` (480p/720p),
`video_style: continuous`, and `aspect_ratio: match|16:9|9:16`. Standard/HD planning
uses 1×/2× billable duration; confirm actual account pricing before approval.
`prepare_video` stops before rendering; `plan_render → start_render → get_render`
uses an explicit approved plan and idempotency key. Output is under
`project.video_url`. Studio starts use `confirm_generation: true` and durable
idempotency keys. These are source-reviewed contracts, not completed customer
client tests.

For support, use the public [contact page](https://www.bhuman.ai/contact).
Share sanitized job references, never credentials or private recipient data.

LeadR uses the hosted MCP connection and MCP key, with `leadr_*` tools for owned campaign drafts, plans, no-send previews, separately approved outreach and operation recovery. It is a restricted rollout; verify live tool discovery and account eligibility. Do not infer customer access from private source or confuse LeadR campaign IDs with Studio campaign IDs.
