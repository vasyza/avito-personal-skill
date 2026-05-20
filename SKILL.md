---
name: avito-personal-cli
description: "Operate a compiled Avito personal-account CLI binary. Use when the user asks an agent to work with Avito through an existing avito executable: import a browser-cookie session from copied curl/cookie exports, check session status, list dialogs, read dialog messages, create text/Markdown transcripts, fetch listing info, or send messages with explicit confirmation. Do not use this skill to change the executable."
---

# Avito Personal CLI

## Scope

Use only the compiled `avito` binary. Treat the binary as the only interface.

If the binary path is not provided, locate it in this order:

1. `AVITO_CLI_BIN`
2. `avito` from `PATH`
3. `avito-personal-cli` from `PATH`
4. `~/.local/avito/avito`
5. `~/.local/avito/avito-personal-cli`
6. `./avito`
7. `./avito-personal-cli`

After locating the executable, set `AVITO_BIN` to that path for the commands below.

Users normally install the binary globally:

```sh
git config --global url."ssh://git@github.com/".insteadOf "https://github.com/"
GOPRIVATE=github.com/vasyza/avito-personal-cli \
GONOPROXY=github.com/vasyza/avito-personal-cli \
GONOSUMDB=github.com/vasyza/avito-personal-cli \
go install github.com/vasyza/avito-personal-cli@latest
```

The CLI source repository is private. The user must have GitHub access to `vasyza/avito-personal-cli`, and their SSH key must work with GitHub. The private-module environment variables are required so Go does not query the public module proxy or checksum database for the private module. This install creates a binary named `avito-personal-cli`.

If no binary is found, ask the user to install it or provide `AVITO_CLI_BIN`.

Use the binary's default session path unless the user provides `--session`. The default lives under the user's config directory, for example `~/.config/avito-cli/session.json` on Linux or macOS.

Keep copied curl commands, cookie exports, session files, and raw Avito output in ignored local paths such as `/tmp/avito/`.

## Basic Workflow

Prepare a local workspace:

```sh
mkdir -p /tmp/avito
```

Import a session from a copied browser curl command, a single `Cookie:` header line, or a Netscape cookie export:

```sh
"$AVITO_BIN" session import --cookie-file /tmp/avito/curl.txt
```

Check the session:

```sh
"$AVITO_BIN" session check
```

Show redacted session metadata:

```sh
"$AVITO_BIN" session show
```

## Common Commands

List dialogs:

```sh
"$AVITO_BIN" dialogs list --limit 20
```

Get one dialog:

```sh
"$AVITO_BIN" dialog get '<channelId>'
```

Read messages as JSON:

```sh
"$AVITO_BIN" dialog messages '<channelId>' --limit 50
```

Read messages as text:

```sh
"$AVITO_BIN" dialog messages '<channelId>' --limit 50 --format text
```

Write a Markdown transcript:

```sh
"$AVITO_BIN" dialog transcript '<channelId>' --limit 100 --output /tmp/avito/transcript.md
```

Fetch listing info:

```sh
"$AVITO_BIN" listing get '<itemId-or-url-or-path>'
```

Send a message:

```sh
"$AVITO_BIN" dialog send '<channelId>' --text 'message'
```

The binary should prompt before sending. Use `--yes` only when the user explicitly asks for non-interactive sending:

```sh
"$AVITO_BIN" dialog send '<channelId>' --text 'message' --yes
```

## Safety

- Never commit session files, cookies, copied curl commands, or raw Avito responses.
- Do not print full cookies or session JSON in final answers.
- Do not send messages unless the user explicitly requested sending and the exact message text is known.
- Prefer read-only checks first: `session check`, `dialogs list`, `dialog get`, `dialog messages`.
- If a transcript reports `Has more: true`, say the transcript is incomplete and rerun with a higher `--limit` if needed.

## Reporting Results

Summarize command results in plain language. Link to generated local artifacts when useful, for example:

```text
/tmp/avito/transcript.md
```

For dialogs/messages, report:

- matched channel id;
- listing title if present;
- message count;
- whether `Has more` is true;
- transcript output path if created.

Do not include private cookie values or unnecessary raw JSON in the final response.
