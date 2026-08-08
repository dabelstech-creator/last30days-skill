---
module: tvos-auth-trigger-orchestration
category: docs/solutions/architecture
tags:
  - tvos
  - auth0
  - passkeys
  - llm-triggers
  - shortcuts
problem_type: architecture-pattern
---

# tvOS App Architecture for Identity-Gated LLM Flows

## Problem

tvOS sign-in surfaces should expose only the authentication actions that a backend or LLM policy decision allows, while still sharing the same identity layer as iOS and supporting enterprise sign-in, passkey rebinds, and companion-device automation.

## Recommended Architecture

### Shared identity layer

- Use the Auth0 SDK for tvOS so the Apple TV app follows the same OIDC/OAuth patterns as the iOS app.
- Configure Microsoft Entra ID as an enterprise connection in Auth0, rather than embedding Entra-specific logic in the tvOS client.
- Treat passkey support as a companion-device flow: tvOS can initiate or display the decision, but the iOS companion app should handle passkey creation, assertion, or rebind steps when platform UX is limited.

### LLM trigger orchestration

- Run the system prompt and trigger-policy evaluation before the user reaches the login screen.
- Return machine-readable decision tokens such as `AUTHENTICATE_WITH_PASSKEY`, `REBIND_PASSKEY`, `LOGIN_WITH_ENTERPRISE`, `SIGN_UP`, `RESET_PASSWORD`, and `LOGOUT`.
- Render only the flows allowed by the returned token set so the tvOS UI cannot present actions that policy has denied.

### tvOS SwiftUI components

- Build the auth menu with large, focusable SwiftUI buttons for Login, Logout, Sign-Up, and Reset Password.
- Optimize menu navigation for the Apple TV remote and focus engine, not for pointer or touch-first layouts.
- Include a validation popover or interstitial that shows the trigger decision in a concise, user-safe form before continuing to the selected auth flow.

### Automation integration

- Link an iOS Shortcut as a companion automation for passkey and recovery flows that are easier to complete on iPhone.
- Point both the tvOS app and the iOS Shortcut at the same backend decision endpoint so policy, logging, and token generation stay centralized.

## Implementation Notes

- Keep the trigger decision endpoint deterministic and auditable: response tokens should be stable enum values, not free-form prose.
- Validate tokens server-side again before starting Auth0 flows; the tvOS UI should be a constrained presentation layer, not the policy authority.
- Log the selected token, Auth0 connection, and companion-app handoff outcome without recording secrets, passkey material, cookies, or access tokens.
- Document any new environment variables or per-client installation steps in `CONFIGURATION.md` before shipping the feature.
