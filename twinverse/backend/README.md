# TwinVerse backend

TypeScript backend for student encounters, Faculty Studio, AI feedback and optional voice/document workflows. This supersedes the [initial single-table milestone](PATIENTS-STEP-1-HISTORY.md).

## Start

Requires Node.js 24+, the existing AWS event resources, and authorized Cognito access. See the [root quickstart](../../README.md) and [AWS profile instructions](../aws/README.md).

```sh
npm ci --ignore-scripts
cp .env.server.example .env.server
npm run serve:student
```

The server binds to `127.0.0.1:8787`. Use it behind the frontend's Vite proxy at `http://127.0.0.1:8443`; opening the API port is not the sign-in flow. The start script loads `.env.server` using Node's environment-file support. Existing exported environment values take precedence.

## Configuration

| Setting | Purpose |
| --- | --- |
| `ENABLE_STUDENT_DEVELOPMENT_PREVIEW` | Enabled by the start script; preview requests also require `mode=development-preview`. |
| `ENABLE_FACULTY_DEVELOPMENT` | Set `true` for local faculty routes; Cognito Faculty membership is still required. |
| `ENABLE_PATIENT_FILES` | Defaults to false in the example; enable only with reviewed private storage provisioned. |
| `ELEVENLABS_API_KEY` | Optional server-only voice credential; an unresolved secret placeholder is rejected. |
| `ELEVENLABS_VOICE_ID` | Optional voice identifier for spoken replies. |
| `TWINVERSE_CHALLENGE_PACKAGE` | Optional local path for the legacy CSV/ZIP import scripts. |

AWS clients select the named `twinverse-event` profile and event resource configuration in source. Cognito identifiers are currently fixed in `lib/student-auth.ts`. Merely setting `AWS_PROFILE`, `AWS_REGION`, or `COGNITO_*` variables does not retarget these clients. Use your own authorized session; never commit credential files.

The voice example contains a Secrets Manager reference, not a usable API key. Resolve it through your authorized server-side secret workflow (the event setup uses `asm-exec`) before launching, or leave voice unconfigured. Never put it in a frontend `VITE_*` variable. Text conversations work independently of voice configuration.

## Main modules

| Module | Responsibility |
| --- | --- |
| `scripts/student-preview-server.ts` | Local HTTP routing, request boundaries, student sessions and feedback endpoints |
| `lib/student-auth.ts` | Cognito PKCE flow, token verification and role/session checks |
| `lib/faculty-routes.ts`, `lib/faculty-patients.ts` | Faculty authoring, draft revisions, review and publishing |
| `lib/preview-conversation.ts`, `lib/published-context.ts` | Grounded dialogue and versioned patient context |
| `lib/student-encounters.ts` | Student-owned persistent encounters, revisions and idempotency |
| `lib/encounter-guide.ts`, `lib/encounter-feedback.ts` | Answer guides, rubric evaluation and evidence validation |
| `lib/student-speech.ts`, `lib/student-transcription.ts` | Optional voice output/input |
| `lib/patient-files.ts` | Feature-gated private documents and release checks |

## Validate

```sh
npm run check
npm run lint
npm test
```

Tests use local fixtures and mocked cloud/provider dependencies. The ZIP-import tests build their own synthetic archive, preserving the CSV/portrait consistency checks without requiring a private Downloads path. The authentication test uses a temporary loopback HTTP listener.

## Infrastructure and import utilities

CDK entrypoints are in `bin/`; they define resources but are not invoked by `serve:student`. Do not run deployment/import write commands against an existing event account without reviewing their scope. Historical deployment observations are not a claim about today's live resource state.

```sh
# Prepare an import locally; does not call AWS.
npm run preview:wyatt -- /path/to/challenge.zip
```

Without an argument, legacy imports use `TWINVERSE_CHALLENGE_PACKAGE` or `sources/Challenge 1_ The Virtual Patient Encounter System.zip`. The archive is not committed. See [PATIENT-LOADER.md](PATIENT-LOADER.md), [HUGGINGFACE-IMPORT.md](HUGGINGFACE-IMPORT.md), and the historical milestone for the underlying data contracts.
