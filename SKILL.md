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
3. `~/.local/avito/avito`
4. `./avito`

Users normally install the binary globally:

```sh
go install github.com/vasyza/avito-personal-cli@latest
```

If no binary is found, ask the user to install it or provide `AVITO_CLI_BIN`.

Use a local session path unless the user provides one:

```sh
~/.local/avito/session.json
```

Keep copied curl commands, cookie exports, session files, and raw Avito output in ignored local paths such as `~/.local/avito/`.

## Basic Workflow

Prepare a local workspace:

```sh
mkdir -p ~/.local/avito/out
```

Import a session from a copied browser curl command, a single `Cookie:` header line, or a Netscape cookie export:

```sh
"$AVITO_BIN" --session ~/.local/avito/session.json session import --cookie-file ~/.local/avito/curl.txt
```

Check the session:

```sh
"$AVITO_BIN" --session ~/.local/avito/session.json session check
```

Show redacted session metadata:

```sh
"$AVITO_BIN" --session ~/.local/avito/session.json session show
```

## Common Commands

List dialogs:

```sh
"$AVITO_BIN" --session ~/.local/avito/session.json dialogs list --limit 20
```

Get one dialog:

```sh
"$AVITO_BIN" --session ~/.local/avito/session.json dialog get '<channelId>'
```

Read messages as JSON:

```sh
"$AVITO_BIN" --session ~/.local/avito/session.json dialog messages '<channelId>' --limit 50
```

Read messages as text:

```sh
"$AVITO_BIN" --session ~/.local/avito/session.json dialog messages '<channelId>' --limit 50 --format text
```

Write a Markdown transcript:

```sh
"$AVITO_BIN" --session ~/.local/avito/session.json dialog transcript '<channelId>' --limit 100 --output ~/.local/avito/out/transcript.md
```

Fetch listing info:

```sh
"$AVITO_BIN" --session ~/.local/avito/session.json listing get '<itemId-or-url-or-path>'
```

Send a message:

```sh
"$AVITO_BIN" --session ~/.local/avito/session.json dialog send '<channelId>' --text 'message'
```

The binary should prompt before sending. Use `--yes` only when the user explicitly asks for non-interactive sending:

```sh
"$AVITO_BIN" --session ~/.local/avito/session.json dialog send '<channelId>' --text 'message' --yes
```

## Safety

- Never commit `.local/`, session files, cookies, copied curl commands, or raw Avito responses.
- Do not print full cookies or session JSON in final answers.
- Do not send messages unless the user explicitly requested sending and the exact message text is known.
- Prefer read-only checks first: `session check`, `dialogs list`, `dialog get`, `dialog messages`.
- If a transcript reports `Has more: true`, say the transcript is incomplete and rerun with a higher `--limit` if needed.

## Reporting Results

Summarize command results in plain language. Link to generated local artifacts when useful, for example:

```text
~/.local/avito/out/transcript.md
```

For dialogs/messages, report:

- matched channel id;
- listing title if present;
- message count;
- whether `Has more` is true;
- transcript output path if created.

Do not include private cookie values or unnecessary raw JSON in the final response.
