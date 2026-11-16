---
sidebar_position: 1
title: 'Connect to a remote participant'
description: 'Connect to a participant that requires a token, and set defaults'
---

Use this guide to trace updates on a participant that requires a token, for example a validator you do not run yourself. You do not need the example ledger to be running.

Pass the participant's JSON Ledger API URL, the party to read as, and a bearer token. Use `--token-file` to keep the token out of your shell history:
```bash
dpm trace <update-id> \
  --submitter https://participant.example.com \
  --read-as '<party-id>' \
  --token-file ./token.txt
```
Use `--token <token>` to pass the token on the command line.

## Find an update id

- `dpm trace submit` prints the update id of the command it sends.
- For a command your app sent, look up its command id: `dpm trace --command-id <command-id> --act-as '<party-id>'`. See [Find a command by its id](./find-a-command-by-its-id.md).
- Many app backends log the update id, or the command id, after each submission.

## Get a token

`dpm trace` sends the token as a bearer token and does not fetch one itself. Get it from the identity provider the participant uses. On LocalNet or the QuickStart with OAuth2, that is the bundled Keycloak: request a token for your app's client with the OAuth2 client credentials flow and save the `access_token` field to `token.txt`. On a hosted validator, ask its operator for a token.

## Check the user's rights

The token belongs to a ledger user, and the user needs rights for the party you pass:

| Command | Right |
| --- | --- |
| `dpm trace <update-id>`, `compare`, `--command-id` | `CanReadAs` or `CanActAs` for the `--read-as` party |
| `submit`, `prepare` | `CanActAs` for the `--act-as` party |

The HTTP status code tells you which problem you have:

- HTTP 401: the token is missing, invalid or expired. Get a new token.
- HTTP 403: the token is valid, but the user lacks the right for the party. Check the user's rights.

To check them, send two requests to the participant's JSON Ledger API with the same token. The first returns the user id the token belongs to, the second lists that user's rights:
```bash
curl -H "Authorization: Bearer $(cat token.txt)" \
  https://participant.example.com/v2/authenticated-user

curl -H "Authorization: Bearer $(cat token.txt)" \
  https://participant.example.com/v2/users/<user-id>/rights
```
Look for `CanReadAs` or `CanActAs` with the party you pass to `--read-as`.

## Set defaults

To stop repeating the options, put them in a `.dpm-trace.json` file in your project directory:
```json
{
  "ledgerUrl": "https://participant.example.com",
  "readAs": "<party-id>",
  "tokenFile": "<path-to-token-file>"
}
```
`dpm trace` looks for the file in the current directory and its parents, and stops at the project root, the first directory that has a `.git` or a `daml.yaml`. Use `--config <file>` to point to a different file. `ledgerUrl` replaces `--submitter`, `readAs` replaces `--read-as`, and `token` and `tokenFile` replace `--token` and `--token-file`. Do not commit a file that holds a token.

Each setting also has an environment variable: `DPM_TRACE_LEDGER_URL`, `DPM_TRACE_READ_AS`, `DPM_TRACE_TOKEN` and `DPM_TRACE_TOKEN_FILE`. A flag wins over an environment variable, which wins over the config file.

With the settings in place, the command is shorter:
```bash
dpm trace <update-id>
```

## Wait for an update on another participant

When you trace an update on a different participant than the one that took the submission, the update may not have arrived yet. Add `--wait <seconds>` to retry until it is visible. Here `$LEDGER2` is the second participant's JSON Ledger API URL and `$BOB` is a party it hosts, as set by the example ledger's `up.sh`:
```bash
dpm trace <update-id> --submitter "$LEDGER2" --read-as "$BOB" --wait 10
```
