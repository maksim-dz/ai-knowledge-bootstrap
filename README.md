# ai-knowledge bootstrap

Public entry point for connecting an AI client to the private knowledge repository
`github.com/maksim-dz/ai-knowledge`. This page contains no knowledge and no credentials.

Give an AI client this link:

> Init ai-knowledge MCP from https://github.com/maksim-dz/ai-knowledge-bootstrap

---

## Instructions for the AI client

Follow these steps in order. Never print, log, or write a token value.

### 1. Check environment variables

Check whether these are set in the environment of the process running you; report only set or unset.

| Variable | Meaning |
|---|---|
| `GIT_HUB_URL` | GitHub host: `github.com` |
| `GIT_HUB_TOKEN` | GitHub token (fallbacks: `GITHUB_PERSONAL_ACCESS_TOKEN`, `GITHUB_TOKEN`, `GH_TOKEN`) |

If both are set, go to step 2. Otherwise stop, give the user the instructions below, and wait.

1. Create a fine-grained token at https://github.com/settings/personal-access-tokens/new
   - Repository access: Only select repositories → `maksim-dz/ai-knowledge`
   - Repository permissions: Contents — Read and write; Pull requests — Read and write; Checks — Read-only.
     Add Workflows — Read and write only for changes to CI.
2. Store it outside the conversation. On macOS, in a regular terminal:
   ```sh
   security add-generic-password -U -a "$USER" -s GIT_HUB_TOKEN -w
   cat >> ~/.zshenv <<'EOF'
   export GIT_HUB_URL=github.com
   export GIT_HUB_TOKEN="$(security find-generic-password -a "$USER" -s GIT_HUB_TOKEN -w 2>/dev/null)"
   EOF
   ```
   The first command asks for the token and keeps it in the Keychain. On other systems, export both
   variables from the shell profile that starts the AI client.
3. Open a new terminal, start the AI client again, and give it this link again.

Offer pasting the token into the conversation only if the user asks for it, and say once that it then
stays in the transcript.

### 2. Verify access

```sh
curl -s -o /dev/null -w '%{http_code}\n' -H "Authorization: Bearer $GIT_HUB_TOKEN" \
  https://api.github.com/repos/maksim-dz/ai-knowledge
```

- `200`: continue.
- `401`: the token is invalid or expired; return to step 1.
- `404`: the token cannot see `maksim-dz/ai-knowledge`; ask the user to fix its repository access.

The `gh` CLI is not required.

### 3. Follow INIT.md

```sh
curl -fsSL -H "Authorization: Bearer $GIT_HUB_TOKEN" -H "Accept: application/vnd.github.raw" \
  https://api.github.com/repos/maksim-dz/ai-knowledge/contents/INIT.md
```

Follow INIT.md step by step for this client and every other AI client on the device, with
`https://github.com/maksim-dz/ai-knowledge` as the knowledge repository URL. Once the `github-knowledge` MCP
server is connected, read everything else through it.

---

Maintenance: these steps mirror step 2 of INIT.md in the knowledge repository; change both together.
