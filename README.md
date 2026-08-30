# Ory Agent Extension: Gemini CLI

Security and developer experience for [Gemini CLI](https://github.com/google-gemini/gemini-cli), powered by [Ory](https://ory.com).

Gemini runs real actions on your machine — editing files, running shell commands, calling APIs. This plugin gives those actions an identity, a permission check, and an audit trail.

One command installs two independent halves:

- **Developer experience** — the Ory skill catalog, `/ory:` slash commands, the local Ory dev stack, an MCP server, and activity logging. No account, no keys, nothing to configure: it works the moment it's installed.
- **Ory Agent Security** — browser sign-in, brokered permission checks before tool calls, and the delegation trail. Opt-in connection details from the [Ory Console](https://console.ory.sh) switch it on, and it takes nothing away from the half above. See [Connect to Ory Agent Security](#connect-to-ory-agent-security).

## What you'll need

- [Gemini CLI](https://github.com/google-gemini/gemini-cli), installed and signed in
- Node.js **22 or newer**
- [Docker](https://docs.docker.com/get-docker/) — only if you want to run Ory locally
- macOS or Linux (Windows works via WSL2)

## Get started

Run one command It installs the extension:

```bash
npx -y -p @ory/gemini-cli ory-gemini install
```

This registers the extension — its hooks, skills, `/ory:` slash commands, and the bundled Ory MCP server. Confirm everything landed with:

```bash
npx -y -p @ory/gemini-cli ory-gemini status
```

`status` is your one-stop check: what's configured, who's signed in, which tools are covered by permissions, whether the extension and skills registered, and recent activity. Until you connect Agent Security, the identity and permission rows say so and name what's missing.

> Prefer to install the extension natively? Run `gemini extensions install https://github.com/ory/gemini-cli-extension`. That registers the extension the same way, with Agent Security not connected. If the `gemini` binary isn't on your `PATH`, `npx -y -p @ory/gemini-cli ory-gemini-setup` writes the extension config straight into your project's `.gemini/settings.json`.

## Skills and commands

Installing the extension drops the full Ory playbook catalog into Gemini CLI. **Skills** are model-invoked — just say what you want in plain language and the matching one takes over.

| Skill | What it does for you |
|---|---|
| `ory-auth-setup` | Adds a complete auth system to your app — login, registration, recovery, verification, settings — on [Ory Elements](https://github.com/ory/elements) |
| `ory-login-flow` | Builds just the pages, wired to Ory's self-service flows |
| `ory-social-login` | "Sign in with…" for Google, GitHub, Apple, Microsoft, Discord, Slack, GitLab, Facebook |
| `ory-local-dev` | Develops and tests login/permission flows against a local Ory — no project, no account, offline |
| `ory-permissions-onboarding` | Walks a fresh install from observe mode to enforced per-tool permissions without getting blocked |
| `ory-build-agent` | Drops `@ory/argus` into an agent *you* own — Claude Agent SDK, OpenAI Agents, Mastra, Vercel AI, LangGraph/PydanticAI |
| `ory-build-integration` | Wires Ory into your app: Action webhooks, JWT validation at a gateway, live event streams |
| `ory-contribute-integration` | Authors and submits an integration to the public `ory/integrates` registry |
| `ory-e2b-sandbox` | Scaffolds an E2B sandbox template that boots with this plugin preinstalled |
| `ory-temporal-worker` | Scaffolds a Temporal TypeScript worker where every Activity is authenticated, authorized, and audited |

**Slash commands** run the local stack directly:

| Command | What it does |
|---|---|
| `/ory:local-up` | Starts a local Ory (Identities, OAuth2, Permissions) in Docker and seeds a test user — it prints the email + password to sign in with |
| `/ory:local-down` | Stops it, keeping your data volumes |
| `/ory:temporal-up` | Starts a local Temporal dev server for the `ory-temporal-worker` scaffold |

The local stack runs entirely on your laptop: Ory APIs at `http://localhost:4000`, a login UI on `:4455` (not `:3000`, to dodge Next.js port clashes), and the Ory Console on `:4100`.

A built-in **Ory MCP server** rounds it out — Gemini can manage identities, projects, and permissions straight from chat.

So: ask Gemini *"add Ory login to this app"* and it scaffolds the pages, starts a local Ory, and wires them together.

## What you get

Out of the box, every tool Gemini runs produces a privacy-safe structured activity event in the unified local log.

Once you connect to Ory Agent Security, two more things happen automatically:

- **Who's driving.** You sign in once in your browser; each Gemini session and typed sub-agent gets its own identity and registers automatically on first use. No tokens to copy around, and the "who acted on whose behalf" trail stays queryable later.
- **What it's allowed to do.** Before a tool runs, Ory checks whether it's permitted. It starts in **observe mode** — nothing is blocked, you just *see* what would be — so it never gets in your way on day one.

If Ory is ever unreachable, the extension gets out of the way and lets Gemini keep working — so it can't lock you out.

### See what's happening

Everything the extension does is observable out of the box:

- **Activity log.** Privacy-safe activity is always appended to `~/.config/ory-agent-plugins/gemini-cli/ory-agent-debug.log`. View events, decisions, and errors live with:

  ```bash
  npx -y -p @ory/gemini-cli ory-gemini watch
  ```

  Set `ORY_AGENT_LOG_FILE` to override the path; set it empty to disable file persistence.
- **Live debug.** Launch Gemini CLI with `ORY_AGENT_DEBUG=true` to add verbose local diagnostics, including raw shell commands, to the watched log and stderr. Secrets are recursively redacted; pass `watch --json` for NDJSON.

### Ready to enforce?

Once connected, the deny posture lives on the Ory project: when the observe-mode activity looks right, an admin promotes it to **enforce** in the **Ory Console** (Agent Security). Every session reads that posture live, so nothing has to be reinstalled.

```bash
npx -y -p @ory/gemini-cli ory-gemini permissions   # what the project grants, and the live mode
```

Then a denied tool is actually blocked and Gemini shows why.

## Connect to Ory Agent Security

Copy the connection details from the [Ory Console](https://console.ory.sh) under **Agent Security**:

| Value | Flag | Environment variable |
|---|---|---|
| Project URL | `--project-url` | `ORY_PROJECT_URL` |
| Agent Security URL | `--agent-security-url` | `ORY_AGENT_SECURITY_URL` |
| Sign-in client id override (default `ory-agent-security-login`) | `--oauth2-client-id` | `ORY_OAUTH2_CLIENT_ID` |

```bash
npx -y -p @ory/gemini-cli ory-gemini configure \
  --project-url https://<slug>.projects.oryapis.com \
  --agent-security-url https://agents.console.ory.com
```

`install` accepts the same flags, so you can register the plugin **and** connect in one shot (`install --project-url <URL> --agent-security-url <URL>`). The login client defaults to `ory-agent-security-login`; custom deployments can override it with `--oauth2-client-id`. Existing configurations fall back to the project URL when the Agent Security URL is unset. To turn sign-in and checks back off later, use `configure --disconnect`.

**Ory Network or OEL.** Either works. For an Ory Network project the URL is `https://<slug>.projects.oryapis.com`; for a self-hosted **Ory Enterprise License** deployment, point `--project-url` at that deployment's base URL and use the sign-in client id from its Agent Security configuration. Everything downstream — sign-in, checks, delegation — is identical.

**There is nothing else for you to create.** The sign-in client, the permission model, the per-tool grants and blocks, and the observe/enforce posture are all provisioned in the Console by someone with project access. At runtime the plugin only *reads* permissions — it has no write path into your project, which is why installing it needs no workspace privilege. Each Gemini session registers its own identity automatically on first use.

Sign-in runs at the start of every session and never blocks — a declined, skipped, or timed-out login simply leaves that session without a user identity.

Settings are saved to `~/.config/ory-agent-plugins/config.json` and shared across all your Ory agent plugins; environment variables win when both are set. Running headless with an OAuth2 access token already? Set `ORY_USER_OAUTH2_TOKEN` and the browser step is skipped entirely.

## Commands

```
ory-gemini install | uninstall        Install (add --project-url to also connect Agent Security) / remove
ory-gemini status                     Show configuration, identities, permission coverage, recent activity
ory-gemini permissions <cmd>          status  (read-only; grants + posture live in the Ory Console)
ory-gemini configure <flags>          Connect a project (--project-url) or --disconnect
ory-gemini agent <status|unregister>  Manage Gemini's own auto-created identity
ory-gemini local <up|down|status|…>   Run / manage a local Ory in Docker
ory-gemini version                    Print plugin, core, and Node versions (--json for machine-readable)
```

All prefixed with `npx -y -p @ory/gemini-cli`.

## Troubleshooting

- **`/ory:local-up` fails** — make sure Docker is running and ports `4000`, `4100`, `4455`, and `16686` are free.
- **Browser sign-in loops** (after connecting) — reset with `ory-gemini agent unregister` and try again.
- **Running an older CLI than expected** — `npx -p @ory/gemini-cli` (no version pin) reuses a previously-downloaded copy instead of re-resolving to the latest. The install banner prints the running version; confirm it with `npx -y -p @ory/gemini-cli ory-gemini version`. To force the current release, clear the cache and reinstall: `rm -rf ~/.npm/_npx` then `npx -y -p @ory/gemini-cli ory-gemini install`. Pinning an exact version (`@ory/gemini-cli@<version>`) also bypasses the cached copy.

## Learn more

- [Ory documentation](https://www.ory.com/docs/) · [Ory Console](https://console.ory.sh) · [Ory Elements](https://github.com/ory/elements)
- [Gemini CLI documentation](https://github.com/google-gemini/gemini-cli)

## License

Apache-2.0
