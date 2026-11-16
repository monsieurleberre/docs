---
sidebar_position: 1
title: 'Connect to a remote participant'
description: 'Get a token and trace updates on a participant that requires one'
---

Use this guide to trace updates on a participant that requires a token, for example a validator you do not run yourself. You do not need the example ledger to be running.

## Get a token

A participant that requires a token checks it on every request to its JSON Ledger API. The token identifies the ledger user making the request. `dpm trace` sends the token you give it as a bearer token and does not fetch one itself, so get one from the identity provider the participant uses:

- On the [Canton Network QuickStart](https://github.com/digital-asset/cn-quickstart) with OAuth2 enabled, the identity provider is the bundled [Keycloak](https://www.keycloak.org/), an open-source identity server. Request a token for your app's client with the OAuth2 client credentials flow, and save the `access_token` field of the response to `token.txt`.
- On a hosted validator, ask its operator for a token.

## Trace an update with the token

Pass the participant's JSON Ledger API URL, the party to read as, and the token. Use `--token-file` to keep the token out of your shell history:
```bash
dpm trace <update-id> \
  --submitter https://participant.example.com \
  --read-as '<party-id>' \
  --token-file ./token.txt
```
Use `--token <token>` to pass the token on the command line.

To set the URL, party and token once instead of on every command, see [Configuration](../reference/configuration.md).

## Check the user's rights

The ledger user the token identifies needs rights for the party you pass:

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
