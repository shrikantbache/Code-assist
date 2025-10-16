Enterprise Code-Assist Extension (Based on Continue) — Implementation Specification

> Purpose: Extend the open-source Continue VS Code extension into a secure, enterprise-level Code Assist platform, integrating existing SSO authentication via the auth_server, and adding features for API key management, model authorization, and an admin dashboard.




---

1. Scope & Goals

Primary goals

1. Integrate the existing auth_server (SSO-based authentication) for user login in both the VS Code extension and dashboard.


2. Allow users to add their App Key (API key) in the extension to access authorized models.


3. Provide an Admin Dashboard for managing API keys, user authorization, and access scopes to different models.


4. Use the auth_server for validating user identity and extend it to support API key issuance and authorization.


5. Enable seamless use of API keys inside the extension for calling enterprise-approved model endpoints.




---

2. High-Level Architecture

Components

Continue VS Code Extension (Client) — Main code-assist interface for developers. Supports login through auth_server and entry of App Key for model access.

Auth Server (Existing) — Already supports SSO via OIDC/SAML. To be extended for API key management endpoints and model access validation.

Admin Dashboard — Web interface for admins to generate/revoke API keys and manage model permissions.

Broker/Model Gateway Service — Handles requests from the extension, validates API keys via auth_server, and proxies requests to configured model providers (OpenAI, Claude, Mistral, etc.).

Database & Storage — For storing hashed API keys, scopes, and usage metrics.


Flow Overview:

1. User signs in using SSO (via auth_server).


2. Admin creates and assigns API keys to users via Dashboard.


3. User adds API key in VS Code extension settings.


4. Extension uses the API key when calling the Broker service.


5. Broker verifies key through the auth_server and forwards the model request to provider.


6. Response is returned back to the extension.




---

3. Authentication & Authorization

3.1 SSO Integration

Reuse the existing auth_server for login validation. The extension opens a browser window for SSO login and receives an access token.

Tokens are validated directly through auth_server's introspection endpoint.

No new SSO flow is required — only reuse and token validation APIs from auth_server.


3.2 API Key Management (Extension to Auth Server)

Extend auth_server with endpoints:

POST /api/keys — Generate new API keys (admin-only).

GET /api/keys — View keys assigned to a user or org.

DELETE /api/keys/{id} — Revoke a key.

POST /api/keys/validate — Validate an API key and return authorized model scopes.


Keys stored as hashed values in DB (e.g., HMAC-SHA256). Only hash and metadata (owner, scopes, expiry) are stored.


---

4. Admin Dashboard

Purpose: Web interface (React or Vue app) integrated with the auth_server to manage API keys and authorizations.

Features

Login through SSO (reuses auth_server tokens).

Generate API keys (show raw key once, store hash only).

Assign scopes (models allowed, e.g., codeassist-basic, codeassist-advanced).

Set quotas and expiry dates for keys.

View usage stats (calls, token counts, last used).

Revoke or rotate keys.


Endpoints Used — /api/keys, /api/users, /api/models, /api/usage (all backed by auth_server).


---

5. Continue Extension Integration

Goal: Make the experience of adding and using an API key seamless for authenticated enterprise users.

UI Changes

SSO Login Button — Uses auth_server for authentication.

App Key Field — Input field in settings to paste user’s API key.

Validate Key Button — Calls /api/keys/validate to confirm validity and fetch available models.

Model Selector Dropdown — Lists models the key has access to (from validation response).

Status Indicator — Shows login state, key expiry, and model in use.


Request Flow

1. User signs in via SSO.


2. Extension prompts user to add App Key (API key).


3. For each model call:

Include Authorization: ApiKey <raw-key> in request header.

Broker calls auth_server /api/keys/validate to confirm key validity.

If valid, broker forwards request to model provider.



4. Broker returns model output to extension.




---

6. Database Schema (Simplified)

Table	Description

users	user_id, email, org_id, role
api_keys	key_hash, user_id, scopes, expiry, revoked
models	model_name, provider_name, config_json
usage_logs	api_key_id, timestamp, model, tokens_used



---

7. Security Considerations

Never store raw API keys; only hashes.

Validate all tokens and keys via existing auth_server APIs.

Encrypt sensitive data (KMS/Vault).

Use HTTPS/TLS for all communication.

Implement rate limiting per key and per org.

Audit all model calls and key creation events.



---

8. Integration APIs (Auth Server Extensions)

Generate Key (Admin)

POST /api/keys
Authorization: Bearer <admin-token>
{
  "user_id": "u123",
  "scopes": ["codeassist-advanced"],
  "expires_at": "2026-01-01"
}

Validate Key (Used by Broker)

POST /api/keys/validate
Authorization: ApiKey <raw-key>
Response:
{
  "valid": true,
  "user": {"id": "u123", "email": "dev@example.com"},
  "scopes": ["codeassist-basic", "codeassist-advanced"]
}


---

9. Deployment & Infrastructure

Deploy auth_server and dashboard as separate services (Docker/Kubernetes).

Broker runs behind API Gateway for secure access.

Database and secrets managed by Vault/KMS.

Continue extension distributed internally through VSIX package.



---

10. Implementation Milestones

Milestone	Deliverable

M1	Extend auth_server with API key endpoints
M2	Implement Broker service for model validation & proxy
M3	Build Admin Dashboard (key generation & management)
M4	Update Continue extension (App Key UI + SSO integration)
M5	Testing, logging, rate limiting, and observability



---

11. Summary

This implementation leverages the existing auth_server for SSO and extends it to manage API keys and model access authorization. The Admin Dashboard centralizes control over key lifecycle and permissions, while the Continue VS Code extension provides a seamless, authenticated user experience for code-assist powered by enterprise-approved LLM models.



I've refined the spec to reflect that Code Assist is a VS Code extension, with SSO handled by the existing Auth Server (shortened section), and clarified how API Key management and dashboard integration extend that service.

Would you like me to now add a sequence diagram or API contract draft (OpenAPI format) for the API key lifecycle and model access flow?


