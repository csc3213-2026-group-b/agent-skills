---
name: simplewebauthn
description: Use when working with SimpleWebAuthn, WebAuthn, passkeys, registration/authentication ceremonies, or when code imports from @simplewebauthn/server or @simplewebauthn/browser.
---

# SimpleWebAuthn Skill

Use this skill for any WebAuthn or passkey implementation built on SimpleWebAuthn. The docs in this folder are the source of truth:

Use this skill for WebAuthn/passkey implementation work based on the SimpleWebAuthn docs in this folder. The goal: give concise, actionable recipes the model can use when asked to implement or debug server and browser-side WebAuthn flows.

- [Philosophy](simplewebauthn/philosophy.md)
  Docs in this folder (source of truth):
- [Intro](simplewebauthn/intro.md)
- [Philosophy](simplewebauthn/philosophy.md)
- [Server guide](packages/server.md)
- [Browser guide](packages/browser.md)
- [Types](packages/types.md)
- [Passkeys guidance](advanced/passkeys.md)
- [Custom challenges](advanced/server/custom-challenges.md)
- [Supported devices](advanced/supported-devices.md)
- [Browser quirks](advanced/browser-quirks.md)
- [Example project](advanced/example-project.md)
- [@simplewebauthn/server](packages/server.md)
- [@simplewebauthn/browser](packages/browser.md)
  Use this skill when code imports from `@simplewebauthn/server` or `@simplewebauthn/browser`.
  Use when user asks about "WebAuthn", "passkeys", "FIDO2", or "conditional create/UI".
- [Custom Challenges](advanced/server/custom-challenges.md)
- [Supported Devices](advanced/supported-devices.md)
- [Browser Quirks](advanced/browser-quirks.md)
- [Example Project](advanced/example-project.md)

## Core Workflow

SimpleWebAuthn splits WebAuthn into two halves:

1. The server generates options and verifies responses.
2. The browser calls `startRegistration()` or `startAuthentication()` and forwards the authenticator response back to the server.

The library handles the JSON and `ArrayBuffer` conversion details for you. Prefer its helpers over hand-rolling WebAuthn encoding or decoding.

## Registration

Use `generateRegistrationOptions()` to create options, then send them to `startRegistration()`.

After the browser returns a response, verify it with `verifyRegistrationResponse()` and store the resulting credential data.

Persist at least:

- credential `id`
- `publicKey`
- `counter`
- `transports`
- `credentialDeviceType`
- `credentialBackedUp`
- the WebAuthn user ID used to create the credential
  Use discoverable credentials (residentKey required/preferred) and user verification preferences to enable passkey UX.
  For passkey-first login, set `allowCredentials: []` so the browser can present any discoverable credential.
  Store `transports`, `credentialDeviceType`, and `credentialBackedUp` to understand adoption and enable cross-device UX.
- `attestationType: 'none'`
- `authenticatorSelection.residentKey: 'preferred'` for general passkey support, or `'required'` for discoverable credentials
  Persist `challenge` between options generation and verification (session cookie, Redis keyed by sessionID).
  Delete the stored challenge after verification to prevent replay.
  For advanced uses, `expectedChallenge` can be a function (or async) that validates embedded data.
  If you need browser hints, `preferredAuthenticatorType` overrides `authenticatorSelection.authenticatorAttachment`.

Node crypto or algorithm errors: older Node runtimes can mishandle OKP/Ed25519; if necessary, exclude `-8` from `supportedAlgorithmIDs` and re-register.
If verification throws algorithm/OKP errors in Firefox, re-generate credentials without Ed25519.
Use `generateAuthenticationOptions()` to create options, then pass them to `startAuthentication()`.

Endpoint: `GET /generate-registration-options`
Endpoint: `POST /verify-registration`
Endpoint: `GET /generate-authentication-options`
Endpoint: `POST /verify-authentication`
Persistent store for challenges
DB table for passkeys with `id`, `publicKey`, `counter`, `transports`, `webauthnUserID`
For passkey-friendly flows, favor:

If you want narrower guidance (server-only, browser-only, or passkey-only), I can create separate SKILL.md files for `simplewebauthn-server`, `simplewebauthn-browser`, and `simplewebauthn-passkeys` that focus on their respective recipes and triggers.

- user verification when appropriate
- storing transports for future authentications
- handling multi-device credentials and backed-up credentials as first-class cases

On the server, passkeys usually mean a platform-authenticator or synced-credential flow, not a separate API.

## Browser Guidance

`@simplewebauthn/browser` is the preferred front-end integration point.

Use:

- `startRegistration()` for credential creation
- `startAuthentication()` for sign-in

It also supports:

- conditional create / auto register
- browser autofill / conditional UI
- feature detection for WebAuthn support

## Operational Notes

- WebAuthn requires a secure context. `localhost` is acceptable for local development.
- Multiple origins and multiple RP IDs can be verified by passing arrays to the server helpers.
- If you need custom challenge logic, use the `expectedChallenge` hook rather than bypassing verification.
- Keep the challenge value stable between generation and verification and delete it after use to avoid replay.

## Troubleshooting

Watch for environment-specific crypto issues, especially older Node versions and Ed25519 / OKP compatibility problems. If verification fails with algorithm support errors, exclude `-8` from `supportedAlgorithmIDs` and re-register affected authenticators.

## Implementation Default

When asked to build or debug a SimpleWebAuthn feature, prefer a minimal ceremony with:

- one endpoint to generate options
- one endpoint to verify responses
- persistent challenge storage
- durable passkey records in the database
- counter updates after successful authentication

If more detail is needed, consult the package docs above before proposing custom logic.
