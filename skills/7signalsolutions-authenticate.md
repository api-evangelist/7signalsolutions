---
name: 7signal-authenticate
description: Obtain and reuse a 7SIGNAL Platform API bearer token with the OAuth 2.0 client-credentials grant.
api: 7SIGNAL Platform API (Gateway v2)
base_url: https://api-v2.7signal.com
operations:
  - oauth2-token-endpoint
generated: '2026-09-05'
method: generated
source: openapi/7signalsolutions-openapi.json + https://github.com/7Signal/API-Examples/blob/develop/docs/01-authentication.md
---

# Authenticate against the 7SIGNAL Platform API

Every operation on the gateway except the token endpoint itself requires a bearer token.

## Prerequisites

An API Key and API Secret, created in the platform dashboard at <https://start.7signal.com> under
**Users → API Keys → Add**, scoped to an Organization, a Role and a Sapphire Group. In OAuth terms the
API Key **is** the `client_id` and the API Secret **is** the `client_secret` — 7SIGNAL's docs say this
outright.

## Steps

1. `POST https://api-v2.7signal.com/oauth2/token` (`oauth2-token-endpoint`) with
   `Content-Type: application/x-www-form-urlencoded` and the body fields `grant_type=client_credentials`,
   `client_id=<API Key>`, `client_secret=<API Secret>`.
2. Read `access_token`, `token_type`, `expires_in` and `scope` from the JSON response.
3. Send `Authorization: Bearer <access_token>` on every subsequent request.

## Rules

- **The token lives exactly 24 hours (86400s) and the lifetime cannot be changed.** Cache it and reuse
  it for the whole session. Do not request a new token per call — that is the single most common
  mistake against this API and the fastest way to hit the rate limiter.
- There is no refresh token on this grant. When it expires, repeat step 1.
- `401` means the token is missing, expired or invalid. `403` means the token is fine but the key's
  Role/Organization/Sapphire Group does not grant the operation — re-issue the key, do not retry.
- A token endpoint failure returns an RFC 6749 §5.2 error object. `invalid_grant` almost always means
  `grant_type` was missing or was not `client_credentials`.
- Treat a stored token like a password; anyone holding it can call the API as you.
- The MCP server at `https://mcp-v2.7signal.com/mcp` is a **different** authorization surface —
  authorization-code + PKCE, not client credentials. A gateway token will not open it.

## Reference implementation

`auth_utils.py` in <https://github.com/7Signal/API-Examples> does exactly this, with token caching.
