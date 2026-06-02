# Ory for Gemini CLI

This Gemini CLI session has the Ory extension installed. It ships skills for
scaffolding Ory into the user's project, slash commands for driving a local
Ory stack, and lifecycle hooks that authenticate, authorize, and audit every
tool call.

## When to invoke the skills

- **`ory-auth-setup`** — full project setup: install the Ory CLI, create or
  reuse a project, add Ory Elements, configure the SDK, build the login /
  registration / recovery / verification / settings pages, wire session
  middleware. Use this when the user says "add Ory auth to this app."
- **`ory-login-flow`** — just the auth pages with Ory Elements (Next.js App
  Router or React SPA). Use when auth is already partially set up.
- **`ory-social-login`** — Google, GitHub, Apple, Microsoft, Discord, and
  other OIDC providers with Jsonnet data mappers.
- **`ory-local-dev`** — drive the local Ory stack from within the session to
  prototype and test without a remote project.
- **`ory-permissions-onboarding`** — bootstrap permission tuples for the
  built-in tools, switch between observe and enforce mode, troubleshoot
  denials.

## Slash commands

- `/ory:local-up` — start a local Ory instance in Docker (Identities, OAuth2,
  Permissions, plus a login UI on `:3000` and Jaeger on `:16686`, reachable
  through `http://localhost:4000`). Prints seeded test-user credentials.
- `/ory:local-down` — tear it all down.

## Permission mode

After install the extension runs in **observe** mode by default: permission
denies are recorded as audit spans but tools still execute. Tell the user to
run `npx -y -p @ory/gemini-cli ory-gemini permissions enforce` once their
permission set is correct.

If a tool call *is* blocked, the denial reason names the missing relation.
Suggest `npx -y -p @ory/gemini-cli ory-gemini permissions bootstrap` (grants
`use` on every built-in tool to the current user) or ask the Ory MCP server
to write a specific relation tuple.

## Diagnostics

- `npx -y -p @ory/gemini-cli ory-gemini status` — show configuration and
  installation state.
- Set `ORY_AGENT_DEBUG=true` and `ORY_AGENT_LOG_FILE=~/.config/ory-agent-plugins/ory.log`
  to capture structured logs.
