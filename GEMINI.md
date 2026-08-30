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
  Permissions, plus a login UI on `:4455` (not :3000, to avoid Next.js port conflicts), reachable
  through `http://localhost:4000`). Prints seeded test-user credentials.
- `/ory:local-down` — tear it all down.

## Permission mode

After install the extension runs in **observe** mode by default: permission
denies are recorded as activity events but tools still execute. The mode is a
property of the Ory project, read on every session — tell the user that someone
with access to the project promotes it to **enforce** in the Ory Console (Agent
Security) once the permission set is correct. There is no CLI command that sets
it; the plugin only reads it.

If a tool call *is* blocked, the denial reason names the missing relation.
Suggest `npx -y -p @ory/gemini-cli ory-gemini permissions` to see exactly
which tools lack a grant, then have the grant added in the Ory Console (or via
the Ory MCP server, which can write relations on the user's behalf).

## Diagnostics

- `npx -y -p @ory/gemini-cli ory-gemini status` — show configuration and
  installation state.
- Privacy-safe activity is always appended to the default
  `~/.config/ory-agent-plugins/gemini-cli/ory-agent-debug.log`; tail it with `jq`.
- Launch Gemini CLI with `ORY_AGENT_DEBUG=true` for the complete live JSON stream
  on stderr plus verbose local diagnostics. Secrets are recursively redacted.
