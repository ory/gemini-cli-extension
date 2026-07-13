# Ory Agent Extension: Gemini CLI

Security and developer experience for [Gemini CLI](https://github.com/google-gemini/gemini-cli), powered by [Ory](https://ory.com).

**Security.** Gemini runs real actions on your machine — editing files, running shell commands, calling APIs. The extension gives every session a verifiable identity (you sign in once; Gemini and any sub-agents it spawns each get their own), checks every tool call against permissions you control, and records each decision as an audit trace you can ship to your observability stack. It starts in watch mode so nothing is blocked on day one, and if Ory is ever unreachable it steps aside rather than locking you out.

**Developer experience.** A single command installs the extension and walks you through connecting — choose Ory Network, a local Docker stack, or audit-only, and it wires up the project, sign-in client, login, and permissions for you. It also helps you build Ory into your own app: ask in plain language to scaffold login, registration, and recovery pages, run a local Ory, or manage identities and permissions through the bundled MCP server.

## What you'll need

- [Gemini CLI](https://github.com/google-gemini/gemini-cli), installed and signed in
- Node.js **22 or newer**
- [Docker](https://docs.docker.com/get-docker/) — only if you want to run Ory locally
- macOS or Linux (Windows works via WSL2)

## Get started

Run one command. It installs the extension and walks you through connecting:

```bash
npx -y -p @ory/gemini-cli ory-gemini install
```

This registers the extension — its hooks, skills, `/ory:` slash commands, and the bundled Ory MCP server — then asks how you want to connect. **Press Enter for the default**:

- **Ory Network** *(default)* — sign in, or create a free account, in your browser. The project, keys, permissions, and login are all set up for you. Nothing to configure by hand.
- **Local** — run a complete Ory on your laptop with Docker. No account, no signup, no keys. Great for trying it out.
- **Audit-only** — skip Ory entirely and just log what Gemini does.

That's it. Confirm everything landed with:

```bash
npx -y -p @ory/gemini-cli ory-gemini status
```

`status` is your one-stop check: what's configured, who's signed in, which tools are covered by permissions, whether the extension and skills registered, and recent activity. Anything not set up yet shows as `(unset)`.

> Prefer to install the extension natively? Run `gemini extensions install https://github.com/ory/gemini-cli-extension`. That registers the extension but skips the guided setup — run `ory-gemini install` (or `configure`) in a terminal afterwards to connect. Re-run install with `--reconfigure` to change your connection later, or `--no-configure` to skip the wizard. If the `gemini` binary isn't on your `PATH`, `npx -y -p @ory/gemini-cli ory-gemini-setup` writes the extension config straight into your project's `.gemini/settings.json`.

## What you get

Once connected, every tool Gemini runs is governed by Ory — three things happen automatically:

- **Who's driving.** You sign in once in your browser; Gemini gets its own identity too, and registers it automatically on first run (no tokens to copy around). The "who acted on whose behalf" trail stays queryable later.
- **What it's allowed to do.** Before a tool runs, Ory checks whether it's permitted. It starts in **watch mode** — nothing is blocked, you just *see* what would be — so it never gets in your way on day one.
- **A record of everything.** Every decision (allowed, denied, skipped) is logged as a trace you can send to Jaeger, Honeycomb, Grafana, or just a file.

If Ory is ever unreachable, the extension gets out of the way and lets Gemini keep working — so it can't lock you out.

### See what's happening

Everything the plugin does is observable out of the box — no configuration required:

- **Status at a glance.** `npx -y -p @ory/gemini-cli ory-gemini status` shows what's configured, who's signed in, how many built-in tools your permissions cover, and the most recent tool-call activity.
- **Live traces.** Every tool call is recorded as an OpenTelemetry-style span. Watch them stream as the agent works:

  ```bash
  npx -y -p @ory/gemini-cli ory-gemini watch
  ```

  Spans are also written to `~/.config/ory-agent-plugins/gemini-cli/ory-agent-trace.ndjson` (NDJSON, one span per line) — tail that file, or point `OTEL_EXPORTER_OTLP_ENDPOINT` at a collector to ship them straight to Jaeger, Honeycomb, or Grafana.
- **Debug log.** For a verbose play-by-play, set `ORY_AGENT_DEBUG=true`; structured logs land in `~/.config/ory-agent-plugins/gemini-cli/ory-agent-debug.log`.

### Ready to enforce?

When the watch-mode logs look right, turn on blocking with one command (setup already granted you the built-in tools):

```bash
npx -y -p @ory/gemini-cli ory-gemini permissions enforce
```

Now a denied tool is actually blocked and Gemini shows why. Go back to watch mode anytime with `permissions observe`. Use `permissions status` to see what's covered and `permissions bootstrap` to (re-)grant the built-in tools — or just ask Gemini in chat, e.g. *"grant me use of the shell tool."*

## Also: add login to your own app

Beyond securing Gemini, the extension helps you build Ory into whatever you're working on. Ask Gemini *"add Ory login to this app"* and it scaffolds the login, registration, recovery, and settings pages (using [Ory Elements](https://github.com/ory/elements)) wired to a local Ory — no signup or keys needed. Start that local Ory with `/ory:local-up` (it prints a test email + password to sign in with) and tear it down with `/ory:local-down`.

Bundled **skills** (just ask in plain language) cover more: `ory-auth-setup`, `ory-login-flow`, `ory-social-login` (Google, GitHub, Apple…), `ory-permissions-onboarding`, and playbooks for wiring Ory into your own agents, E2B sandboxes, or Temporal workers. Real `/ory:` **slash commands** run the local stack (`/ory:local-up`, `/ory:local-down`, `/ory:temporal-up`). And a built-in **Ory MCP server** lets Gemini manage identities, projects, and permissions straight from chat.

The local stack runs a complete Ory on your laptop: the Ory APIs (Identities, OAuth2, Permissions) at `http://localhost:4000`, a login UI on `:4455` (not `:3000`, to avoid Next.js port conflicts), and Jaeger (the trace viewer) on `:16686`. Everything works offline.

## Configure by hand (CI / advanced)

The guided setup covers most people. For scripted or CI setups, or to point at an existing Ory Network project, configure directly. Settings are saved to `~/.config/ory-agent-plugins/config.json` and shared across all your Ory agent plugins and extensions; environment variables win when both are set.

```bash
npx -y -p @ory/gemini-cli ory-gemini configure \
  --project-url https://<slug>.projects.oryapis.com \
  --oauth2-client-id <login client id> \
  --user-login
```

Gemini's own identity registers itself automatically on first run — nothing to create. The `--oauth2-client-id` is the one piece browser sign-in needs, so `configure` won't save a project URL without it; the guided setup makes it for you, or see below to do it by hand. For logging-only with no checks, use `--audit-only`. The matching env vars are `ORY_PROJECT_URL`, `ORY_OAUTH2_CLIENT_ID`, `ORY_USER_LOGIN`, and `ORY_AGENT_API_KEY`.

<details>
<summary>Create the sign-in client by hand</summary>

The guided setup normally does this. To do it yourself, create a **public** OAuth2 client (no secret) listing all four loopback URLs — the extension tries each in turn so sign-in survives a busy port:

- `http://127.0.0.1:47823/callback`
- `http://127.0.0.1:47824/callback`
- `http://127.0.0.1:47825/callback`
- `http://127.0.0.1:47826/callback`

Create it with the [Ory CLI](https://www.ory.com/docs/guides/cli/installation):

```bash
ory create oauth2-client --project <project-id> \
  --name "ory-agent-plugin" \
  --grant-type authorization_code,refresh_token \
  --response-type code \
  --scope openid,offline_access \
  --token-endpoint-auth-method none \
  --redirect-uri http://127.0.0.1:47823/callback \
  --redirect-uri http://127.0.0.1:47824/callback \
  --redirect-uri http://127.0.0.1:47825/callback \
  --redirect-uri http://127.0.0.1:47826/callback
```

…or make one in the [Ory Console](https://console.ory.sh) under *OAuth2* → *Clients* → *Create client* (pick "Public client", "Authorization Code" + "Refresh Token" grants, scopes `openid offline_access`, paste the four URLs). Pass the resulting id to `configure --oauth2-client-id`. Running headless with a session token already? Set `ORY_USER_SESSION_TOKEN` and skip the browser step entirely.

</details>

With nothing configured, the extension still loads and runs in **pass-through mode**: skills, slash commands, and logging work, but no checks run and nothing is blocked. Perfectly fine if you only want the app-building features.

## Commands

```
ory-gemini install | uninstall        Install/remove; --reconfigure re-runs setup, --no-configure skips it
ory-gemini status                     Show configuration, identities, permission coverage, recent activity
ory-gemini watch                      Tail the live trace stream (OTel spans)
ory-gemini permissions <cmd>          status | bootstrap | observe (watch) | enforce (block)
ory-gemini configure <flags>          Point at a project by hand (--project-url, --oauth2-client-id, --user-login, --audit-only)
ory-gemini agent <status|unregister>  Manage Gemini's own auto-created identity
ory-gemini local <up|down|status|…>   Run / manage a local Ory in Docker
```

All prefixed with `npx -y -p @ory/gemini-cli`.

## Troubleshooting

- **`/ory:local-up` fails** — make sure Docker is running and ports `4000`, `4100`, `4455`, and `16686` are free.
- **Browser sign-in loops** — reset with `ory-gemini agent unregister` and try again.
- **`npx` grabbed an old version** — force the latest: `npx -y -p @ory/gemini-cli@latest ory-gemini …`.
- **Want to see what's happening** — `npx -y -p @ory/gemini-cli ory-gemini status` for a snapshot, `npx -y -p @ory/gemini-cli ory-gemini watch` for the live trace stream, or set `ORY_AGENT_DEBUG=true` for a verbose log. Traces and logs live under `~/.config/ory-agent-plugins/gemini-cli/` (see [See what's happening](#see-whats-happening)).

## Learn more

- [Ory documentation](https://www.ory.com/docs/) · [Ory Console](https://console.ory.sh) · [Ory Elements](https://github.com/ory/elements)
- [Gemini CLI documentation](https://github.com/google-gemini/gemini-cli)

## License

Apache-2.0
