This project has the Ory agent plugin installed. Tool invocations are checked against Ory Permissions (Zanzibar-style) and traced automatically.

If a tool call is blocked, the reason will indicate which permission is missing. The user may need to update their Ory Permissions configuration to grant access.

To check configuration status, the user can run: `npx ory-gemini status`
