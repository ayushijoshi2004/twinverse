# TwinVerse frontend

React/TypeScript interface for virtual-patient encounters, faculty case authoring and rubric feedback. The current student interface uses the local backend; it does not silently replace failed API calls with mock conversations.

## Start

Requires Node.js 24+ and npm. Start the API separately using the [root quickstart](../../README.md).

```sh
npm ci --ignore-scripts
npm run dev
```

Vite listens at `http://127.0.0.1:8443` and proxies `/api` to `http://127.0.0.1:8787`.

- `/`: student sign-in, patient selection and saved encounters.
- `/faculty.html`: Faculty Studio; requires a faculty-role session and Cognito Faculty membership.
- `/feedback/faculty`: faculty feedback review and comments.
- Student feedback opens when a session ends or from a saved encounter.

## Sample feedback without AWS

Copy `.env.example` to `.env.local`, set `VITE_FEEDBACK_SOURCE=sample`, restart Vite and open `/feedback/faculty`. Sample comments persist in browser local storage. This does not enable offline encounters or bypass Faculty Studio authentication.

Live feedback is the default. All browser requests use relative `/api` paths; `VITE_API_BASE_URL` is not a supported override. Keep AWS credentials and voice-provider secrets out of all frontend environment variables and bundles.

## Checks

```sh
npm run check
npm run test:integration
npm run build
```

Tests cover API response validation, privacy boundaries, feedback normalization and voice-conversation state. Production assets are written to `dist/`. `npm run preview` serves a build, but does not provide the development API proxy or a production backend.

See [FEEDBACK-INTEGRATION.md](FEEDBACK-INTEGRATION.md) for feedback contracts. Older integration notes describe incremental milestones and should be read alongside the current root/backend READMEs.
