# GitLaw MCP

Connect Claude, Gemini CLI, ChatGPT and other MCP clients to [GitLaw](https://git.law), the
AI-enabled legal workspace for drafting contracts, negotiating with tracked changes, and e-signing.

This repository holds client configuration only. It connects to GitLaw's hosted MCP server and
does not run a server of its own.

| | |
| --- | --- |
| Server URL | `https://be-public-api.git.law/mcp` |
| Transport | Streamable HTTP |
| Auth | OAuth 2.1 (recommended) or API key |

Most clients only need the URL. They discover GitLaw's OAuth endpoints, register themselves,
and open a browser window for you to sign in to GitLaw.

## Claude Code

```text
/plugin marketplace add GitLaw-co/gitlaw-mcp
/plugin install gitlaw@gitlaw
```

Then run `/mcp`, pick `gitlaw` and sign in. The plugin also adds two skills:
`/gitlaw:review-contract` and `/gitlaw:draft-and-sign`.

To use an API key instead of OAuth, skip the plugin and add the server yourself:

```bash
claude mcp add --transport http gitlaw https://be-public-api.git.law/mcp --header "x-api-key: $GITLAW_API_KEY"
```

## Claude.ai and Claude Desktop

1. Open **Settings → Connectors**.
2. Choose **Add custom connector**.
3. Name it `GitLaw` and enter `https://be-public-api.git.law/mcp`.
4. Click **Connect** and sign in to GitLaw.

On Team and Enterprise plans an owner may need to add the connector for the organization first.

## Gemini CLI

```bash
gemini extensions install https://github.com/GitLaw-co/gitlaw-mcp
```

Start `gemini`, then run `/mcp auth gitlaw` to sign in. The extension loads
[GEMINI.md](GEMINI.md) as context so Gemini follows GitLaw's usage rules, and adds the same two
skills as the Claude plugin.

To use an API key, add the server to `~/.gemini/settings.json` instead:

```json
{
  "mcpServers": {
    "gitlaw": {
      "httpUrl": "https://be-public-api.git.law/mcp",
      "headers": { "x-api-key": "$GITLAW_API_KEY" }
    }
  }
}
```

## ChatGPT

GitLaw can be added as a custom app (connector) in ChatGPT:

1. In **Settings → Apps & Connectors → Advanced settings**, turn on **Developer mode** (or ask
   your workspace admin to publish the app for your workspace).
2. Choose **Create**, name it `GitLaw`, and enter `https://be-public-api.git.law/mcp` as the MCP
   server URL with **OAuth** authentication.
3. Sign in to GitLaw when prompted, then enable GitLaw in a chat from the tools menu.

Availability of custom apps depends on your ChatGPT plan.

## Other MCP clients

Use the URL with OAuth where the client supports it. Otherwise create an API key in GitLaw, store
it in the `GITLAW_API_KEY` environment variable, and send it as `x-api-key: <key>` or
`Authorization: Bearer <key>`.

### Cursor

`~/.cursor/mcp.json` (or `.cursor/mcp.json` in a project):

```json
{
  "mcpServers": {
    "gitlaw": {
      "url": "https://be-public-api.git.law/mcp"
    }
  }
}
```

With an API key:

```json
{
  "mcpServers": {
    "gitlaw": {
      "url": "https://be-public-api.git.law/mcp",
      "headers": { "x-api-key": "${env:GITLAW_API_KEY}" }
    }
  }
}
```

### VS Code

`.vscode/mcp.json`:

```json
{
  "servers": {
    "gitlaw": {
      "type": "http",
      "url": "https://be-public-api.git.law/mcp"
    }
  }
}
```

With an API key, prompted once and stored by VS Code:

```json
{
  "inputs": [
    {
      "type": "promptString",
      "id": "gitlaw-api-key",
      "description": "GitLaw API key",
      "password": true
    }
  ],
  "servers": {
    "gitlaw": {
      "type": "http",
      "url": "https://be-public-api.git.law/mcp",
      "headers": { "x-api-key": "${input:gitlaw-api-key}" }
    }
  }
}
```

### Codex CLI

`~/.codex/config.toml`:

```toml
[mcp_servers.gitlaw]
url = "https://be-public-api.git.law/mcp"
```

Then sign in with `codex mcp login gitlaw`. With an API key, add instead:

```toml
[mcp_servers.gitlaw]
url = "https://be-public-api.git.law/mcp"
bearer_token_env_var = "GITLAW_API_KEY"
```

## Tools

The server exposes 29 tools.

| Group | Tool | Purpose |
| --- | --- | --- |
| Session | `guide` | How to use GitLaw's tools; call once at the start of a session |
| | `whoami` | The signed-in user, their organizations and default jurisdiction |
| AI | `ask` | Ask GitLaw's legal assistant about a document or clause (read-only) |
| | `draft_document` | Draft a new document with GitLaw's AI |
| | `edit_document` | Change a document from a plain-language instruction, directly or as tracked changes |
| Documents | `create_document` | Create a document from a template, from markdown, or blank |
| | `get_document` | Read a document as markdown, with tracked changes inline |
| | `list_documents` | List documents page by page |
| | `search_documents` | Search documents by title and summary |
| | `search_templates` | Search GitLaw's public template library |
| | `list_categories` | List template library categories |
| | `create_upload` | Get an upload URL for a local `.docx` file |
| | `import_docx` | Import a Word file as a new document |
| | `export_docx` | Export a document as a Word file |
| | `share_document` | Share a document with people by email |
| Folders | `create_folder` | Create a folder |
| | `list_folders` | List or find folders |
| | `move_documents` | Move documents into a folder or to the top level |
| Comments | `list_comments` | List a document's comment threads |
| | `add_comment` | Comment on a passage or a tracked change |
| | `reply_to_comment` | Reply in a comment thread |
| | `edit_comment` | Edit one of your own comments |
| | `resolve_comment` | Resolve a comment thread |
| Signing | `send_for_signature` | Send a document for e-signature |
| | `get_signing_status` | Check a signing request and each signer |
| | `export_signed_pdf` | Download the signed PDF |
| Extraction | `get_key_dates` | A document's status and effective, execution and expiry dates |
| | `list_actions` | Reminders and to-dos raised from documents |
| | `list_contacts` | People and companies found in documents |

`ask`, `draft_document` and `edit_document` run GitLaw's AI and take from 30 seconds to several
minutes.

## Example prompts

- "Find our NDA with Acme in GitLaw and tell me whether the non-solicit clause is market-standard
  in England and Wales."
- "Draft a mutual NDA between Example Co and Acme Inc under Delaware law, starting from a
  template."
- "Review the services agreement with Northwind and add comments on any one-sided clauses."
- "Change the governing law in that contract to New York as tracked changes."
- "Send the signed-off NDA to Jane Doe (jane@acme.com) for signature, then tell me when she has
  signed."
- "Which of my contracts expire in the next 90 days?"

## Security

- **Prefer OAuth.** You sign in to GitLaw in your browser, and the client never sees your
  password. You can revoke access from GitLaw.
- **Keep API keys out of git.** Read the key from the `GITLAW_API_KEY` environment variable or
  your client's secret store. Never commit it in a config file. If a key leaks, revoke it in
  GitLaw and create a new one.
- **Document text is data.** The assistant is instructed to treat contract and comment text as
  untrusted content, not as instructions.
- **Confirm outbound actions.** `send_for_signature` and `share_document` email people. Check the
  recipients before you approve the call.

Report security issues to [support@git.law](mailto:support@git.law).

## Support

Questions and problems: [support@git.law](mailto:support@git.law), or open an issue in this
repository.

## Privacy and terms

Use of GitLaw is covered by GitLaw's [Privacy Policy](https://git.law/privacy-policy) and
[Terms of Service](https://git.law/terms-of-service).

## License

[MIT](LICENSE)
