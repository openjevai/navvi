# OpenJEV Support

This fork of [navvi](https://github.com/fellowship-dev/navvi) adds optional support for [OpenJEV](https://openjev.sh), a free community gateway to the same Jev model built by [TypeSafe](https://typesafe.ai). TypeSafe stays the default; OpenJEV is an additional option.

## What was added

- `src/input/schema.ts` — `"openjev"` added to the `TRANSPORTS` array (the `deciderTransport` input field); `OPENJEV_API_KEY` recognised in `defaultChooser` and `hasChooserKey`.
- `src/chooser/jev.ts` — `OPENJEV_API_URL` (`https://api.openjev.sh/v1/systemone`) and `OPENJEV_MODEL_ID` (`"openjev"`) constants; `selectProvider` handles the `openjev` provider and the `OPENJEV_API_KEY` env var; `TypeSafeEvaluationModel` accepts `provider` and `label` options so the same class serves both endpoints; `missingCredentialsMessage` mentions `OPENJEV_API_KEY`.
- `src/chooser/index.ts` — exports `OPENJEV_API_URL` and `OPENJEV_MODEL_ID`; `jevReason` names `OPENJEV_API_KEY` when it is the only key set.
- `src/main.ts` — `openjevApiKey` added to `ACTOR_ONLY_KEYS` and `CALLER_KEY_ENV` so the Apify input field maps to `OPENJEV_API_KEY`.
- `src/cli/args.ts` — `--decider-transport` and `--decider` help text updated to mention the `openjev` option and `OPENJEV_API_KEY`.
- `.actor/input_schema.json` — `"openjev"` added to the `deciderTransport` enum; `openjevApiKey` secret field added.
- `tests/input-schema.test.ts` — test expectations updated for the new enum and key.

## Provider selection rule

1. Explicit choice wins: `--decider-transport openjev` (CLI), `deciderTransport: "openjev"` (run input), or `provider: "openjev"` (programmatic) forces the OpenJEV API regardless of other keys.
2. Otherwise, if `AI_GATEWAY_API_KEY` is set → Vercel AI Gateway (unchanged default).
3. Otherwise, if `TYPESAFE_API_KEY` is set → direct TypeSafe API (unchanged).
4. Otherwise, if only `OPENJEV_API_KEY` is set → OpenJEV API.

Anyone with a TypeSafe or Gateway key sees zero behaviour change.

## How to configure

Set the environment variable:

```
export OPENJEV_API_KEY="your-key-from-https://openjev.sh/dashboard"
```

Or pass it via the Apify input field `openjevApiKey`, or force it explicitly:

```
navvi --decider-transport openjev "extract the title and price" <url>
```

## How it was verified

- One live `POST https://api.openjev.sh/v1/systemone` request with the OpenJEV key, model `openjev`, state `ping`, one `noul` question — returned HTTP 200.
- `grep` confirmed no hardcoded `api.typesafe.ai` default was introduced; the existing TypeSafe endpoint is unchanged.

## Upstream

- Original project: https://github.com/fellowship-dev/navvi by @fellowship-dev
- Jev is built by [TypeSafe](https://typesafe.ai). OpenJEV is a community gateway to the same model; it is not affiliated with or endorsed by TypeSafe.
