# ai-knowledge bootstrap

Public entry point for connecting an AI client to the private knowledge repository
`github.com/maksim-dz/ai-knowledge`. This page contains no knowledge and no credentials.

Give an AI client this link:

> Init ai-knowledge MCP from https://github.com/maksim-dz/ai-knowledge-bootstrap

---

## Instructions for the AI client

Follow these steps in order. Never print, log, or write a token value.

### 1. Check environment variables

Check whether this is set in the environment of the process running you; report only set or unset.

| Variable | Meaning |
|---|---|
| `${GIT_HUB_TOKEN}` | GitHub token (fallbacks: `${GITHUB_PERSONAL_ACCESS_TOKEN}`, `${GITHUB_TOKEN}`, `${GH_TOKEN}`) |

`${NAME}` means the environment variable `NAME`: `$NAME` in a POSIX shell, `$env:NAME` in PowerShell.

If it is set, go to step 2. Otherwise stop, give the user the instructions below, and wait.

1. Create a fine-grained token at https://github.com/settings/personal-access-tokens/new
   - Repository access: Only select repositories → `maksim-dz/ai-knowledge`
   - Repository permissions: Contents — Read and write; Pull requests — Read and write; Actions — Read-only.
     Fine-grained tokens have no Checks permission; Actions is what the knowledge workflow reads.
     Add Workflows — Read and write only for changes to CI.
2. Store it outside the conversation, as the environment variable `GIT_HUB_TOKEN`, in a place the
   non-interactive commands of the AI client read:
   - **macOS** (zsh), in a regular terminal. The first command asks for the token and keeps it in the
     Keychain:
     ```sh
     security add-generic-password -U -a "$USER" -s GIT_HUB_TOKEN -w
     cat >> ~/.zshenv <<'EOF'
     export GIT_HUB_TOKEN="$(security find-generic-password -a "$USER" -s GIT_HUB_TOKEN -w 2>/dev/null)"
     EOF
     ```
   - **Linux**: an `export GIT_HUB_TOKEN="..."` line in `~/.zshenv` (zsh) or in `~/.profile` (bash) of
     the login session that starts the AI client. `~/.zshrc` and `~/.bashrc` are not read by
     non-interactive commands.
   - **Windows**, in PowerShell. The command asks for the token without showing it and stores it as a
     user variable:
     ```powershell
     $t = [System.Net.NetworkCredential]::new('', (Read-Host 'GitHub token' -AsSecureString)).Password
     [Environment]::SetEnvironmentVariable('GIT_HUB_TOKEN', $t, 'User'); Remove-Variable t
     ```
     Inside WSL or Git Bash the Linux line applies instead.
3. Fully restart the AI client (from a new terminal where it is started from one) and give it this link
   again.

Offer pasting the token into the conversation only if the user asks for it, and say once that it then
stays in the transcript.

### 2. Verify access

The commands use `${GIT_HUB_TOKEN}` and are written for a POSIX shell; when the token came from a fallback
variable, use that variable instead. In PowerShell write `$env:GIT_HUB_TOKEN` and call `curl.exe`.

Find the account the token belongs to. The login is not a secret; the token value is:

```sh
curl -s -H "Authorization: Bearer ${GIT_HUB_TOKEN}" https://api.github.com/user | grep -m1 '"login"'
```

Check access to the knowledge repository:

```sh
curl -s -o /dev/null -w '%{http_code}\n' -H "Authorization: Bearer ${GIT_HUB_TOKEN}" \
  https://api.github.com/repos/maksim-dz/ai-knowledge
```

In every result other than `200`, tell the user which variable supplied the token and which account login
it belongs to, then the cause and the fix below. Never show the token value or part of it.

- `200`: continue.
- `000`: the request did not reach GitHub, usually a sandbox or network restriction of the AI client.
  Rerun the command with network access before judging the token.
- `401`: the token is invalid, expired, or revoked; no login is returned. Return to step 1.
- `404`: the token works but cannot see `maksim-dz/ai-knowledge`. Name the likely cause:
  - **Wrong account**: the login is not an account with access to `maksim-dz/ai-knowledge`. The variable
    holds another account's token; point it at the right token, open a new terminal, and restart the client.
  - **Repository not selected in a fine-grained token**: the login is right. Open
    https://github.com/settings/personal-access-tokens, edit the token, and under Repository access select
    `ai-knowledge` with the step 1 permissions. A repository that was deleted and recreated must be
    selected again, even under the same name. The token value does not change, so no restart is needed.
  - **Account without access**: a classic token of an account that is not a collaborator. Use a token of
    an account with access, or ask the owner to add that account as a collaborator.

The `gh` CLI is not required.

### 3. Follow INIT.md

```sh
curl -fsSL -H "Authorization: Bearer ${GIT_HUB_TOKEN}" -H "Accept: application/vnd.github.raw" \
  https://api.github.com/repos/maksim-dz/ai-knowledge/contents/INIT.md
```

Follow INIT.md step by step for this client and every other AI client on the device, with
`https://github.com/maksim-dz/ai-knowledge` as the knowledge repository URL. Once the `github-knowledge` MCP
server is connected, read everything else through it.

---

Maintenance: these steps mirror step 2 of INIT.md in the knowledge repository; change both together.
