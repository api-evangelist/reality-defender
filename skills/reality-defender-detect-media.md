---
name: Detect a deepfake in a media file
description: Submit an image, video, audio, or text file to Reality Defender and read back the ensemble deepfake/manipulation verdict.
api: https://docs.realitydefender.com/api-reference/quickstart
base_url: https://api.prd.realitydefender.xyz
auth: X-API-KEY header (key from https://app.realitydefender.ai)
operations:
  - POST /api/files/aws-presigned
  - GET /api/media/users/{request_id}
mcp_tools:
  - reality_defender_generate_upload_url
  - reality_defender_request_file_analysis
  - reality_defender_get_file_info
method: generated
source: https://docs.realitydefender.com/api-reference/quickstart
generated: '2026-07-20'
---

# Detect a deepfake in a media file

Use the Reality Defender detection API to analyze a file for AI generation or
manipulation. Detection is asynchronous: request an upload URL, upload the file,
then poll for the ensemble result.

## Prerequisites
- An API key from the Reality Defender platform (https://app.realitydefender.ai),
  sent on every request as the `X-API-KEY` header.
- Base URL: `https://api.prd.realitydefender.xyz`.
- Prefer an official SDK (`packages/reality-defender-packages.yml`) over raw REST;
  the SDKs handle the upload + polling loop for you.

## Steps
1. **Request a pre-signed upload URL** — `POST /api/files/aws-presigned` with the
   file name. The response contains an upload URL and a `request_id`.
2. **Upload the media** to the returned pre-signed URL (image, video, audio, or
   text). For a social-media link instead, use `POST /api/files/social`.
3. **Poll for the result** — `GET /api/media/users/{request_id}` until the
   analysis completes. Read the **ensemble** field as the overall verdict;
   individual model scores are also returned.
4. **(Optional) submit feedback** on the verdict via the Create User Feedback
   endpoint.

## Rules and conventions
- Authentication: `X-API-KEY` header on every call (see
  `authentication/reality-defender-authentication.yml`).
- Interaction model is submit-then-poll; there are no webhooks. Space out polling
  to avoid `429` (see `conventions/reality-defender-conventions.yml`).
- No idempotency-key contract — do not assume retried submissions are deduplicated.
- Errors are plain HTTP status codes (`401` bad key, `429` rate limit, etc.); see
  `errors/reality-defender-problem-types.yml`.

## Agent access
When driving this from an MCP client, use the official Reality Defender MCP server
(`mcp/reality-defender-mcp.yml`): `reality_defender_generate_upload_url` →
`reality_defender_request_file_analysis` → `reality_defender_get_file_info`.
