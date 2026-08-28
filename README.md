# claude-tools

Small command-line tools that fill gaps in the vendor CLIs Claude Code drives on this machine.
Each tool lives here (versioned) and is exposed through a symlink in `~/bin`, which is on `PATH`.

| Tool   | Purpose |
|--------|---------|
| `conf` | Confluence Cloud search and page reading — the parts `acli confluence` cannot do |

## conf

`acli confluence` only offers `space list` and `page view --id`: no search, no listing by space,
no labels, no write. `conf` talks to the Confluence Cloud REST API directly to cover the rest.
It is **read-only by design** — there is no create or update path, so it cannot damage a space.

### Usage

```bash
conf search <text|CQL> [-s SPACE] [-t TYPE] [-l LIMIT] [--json]
conf page <id> [--raw|--json]
conf children <id> [--json]
conf labels <id>
conf spaces [--json]
conf api <path> [curl args...]
conf help
```

A plain query is wrapped as `text ~ "..."`. A query that already looks like CQL — it contains
`~`, `=`, ` AND ` or ` OR ` — is passed through untouched. `-s`/`-t` are ANDed on top of either
form, and `ORDER BY lastmodified DESC` is always appended.

```bash
conf search "settlement instruction" -s APD -l 20
conf search 'label = "runbook" AND space = "DEV"'
conf search "kubernetes" -s DEV -t page
```

Search prints one tab-aligned row per hit — `id  type  space  title  last-modified` — and that
first column is the page id every other subcommand takes:

```bash
conf page 1635713033        # title + body converted to GitHub-flavored markdown via pandoc
conf page 1635713033 --raw  # the raw storage-format XHTML
conf children 1635713033    # direct child pages: id + title
conf labels 1635713033      # label names, one per line
```

`conf api` is an escape hatch for anything not wrapped, with auth and the base URL already
applied. The path is appended to `https://<site>/wiki`, and extra arguments go to `curl`:

```bash
conf api /api/v2/pages/1635713033
conf api '/rest/api/search?cql=type%3Dpage&limit=5'
```

Both API versions are in play: v2 (`/api/v2/...`) for pages, children, labels and spaces, v1
(`/rest/api/search`) for search, because v2 has no CQL endpoint.

### Configuration

Site and email are read from the acli profile at `~/.config/acli/confluence_config.yaml`, so
there is one place to change accounts. Nothing else is configured in the script.

The API token is looked up in this order:

1. `$CONFLUENCE_API_TOKEN`
2. the file named by `$CONFLUENCE_TOKEN_FILE`
3. `~/.config/acli/confluence-token`

The token acli itself uses is kept in the OS keyring and cannot be read back, which is why
`conf` needs its own copy.

### Installing a token

Confluence and Jira authenticate **separately** even though they share one site. Jira here uses
oauth (`acli confluence auth login --web` would do the same for Confluence), but `conf` needs a
token, so log Confluence in with one:

1. Generate a **plain API token** at
   <https://id.atlassian.com/manage-profile/security/api-tokens>.
   Choose *Create API token* — **not** *Create API token with scopes*. `acli --token` does not
   support scoped tokens and fails with 401/403 even when the token is valid.

2. Log `acli confluence` in with it, reading the token from stdin so it never lands in the shell
   history:

   ```bash
   read -rs TOKEN && printf '%s' "$TOKEN" | acli confluence auth login \
     --site algotrader.atlassian.net --email <you>@wyden.io --token
   ```

3. Give `conf` its own copy, created mode 600 before anything is written to it:

   ```bash
   install -m 600 /dev/null ~/.config/acli/confluence-token
   read -rs TOKEN && printf '%s' "$TOKEN" > ~/.config/acli/confluence-token
   unset TOKEN
   ```

4. Verify both paths:

   ```bash
   acli confluence auth status   # → Authenticated, auth_type: api_token
   conf spaces | head           # → space list
   ```

Never commit the token, and keep it out of this repository — the lookup deliberately uses paths
under `~/.config` rather than anything in the working tree.

### Requirements

`curl`, `jq`, `pandoc` (only for the default markdown rendering of `conf page`), `column`, and an
authenticated `acli confluence` profile for the site and email.

## Installing a tool from this directory

```bash
ln -s ~/Work/Claude/skills/claude-tools/<tool> ~/bin/<tool>
```
