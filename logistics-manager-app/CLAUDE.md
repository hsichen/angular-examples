# LogisticsManagerApp — guide for Claude Code

Angular 21 (standalone + signals) SSR logistics dashboard: fleet tracking, a
Leaflet map, a service queue, and a chat overlay. Express serves both the REST
API and Angular SSR. State flows **UI components → `FleetService` (signal hub) →
`fleet-db.ts` (in-memory store)**; the Express server in `src/server.ts` also
reads `fleet-db.ts` directly.

- Dev server: `ng serve` (http://localhost:4200). Build: `ng build`. Tests: `ng test` (Vitest).
- Data seed and types live in `src/fleet-db.ts`. The central service is `src/app/services/fleet.service.ts`.

## This project is a tutorial — the AI features are NOT built yet

The app ships as a **working dashboard with a deliberately inert AI layer**. There
is no AI SDK in `package.json`; nothing in `src/` calls an LLM. The chat panel
(`src/app/components/chat/chat.ts`) is a **UI shell**: the message area is
hard-coded to an empty state and the send button has no handler.

**The exercise lives in [`codelab.md`](./codelab.md)** — read it first if asked to
"continue" / "pick up" / "add the AI features". It defines three enhancements,
each naming its target files:

1. **AI fleet chat** — add `FleetService.queryFleet(prompt)`, send `units()` state
   to Gemini, then bind the input + message list in `chat.ts`.
2. **AI service prioritization** — in `src/app/components/service-queue/service-queue.ts`,
   watch the `issue` field and have the model classify LOW/MEDIUM/HIGH/CRITICAL.
3. **Predictive diagnostics** — a "Run AI Diagnostic" button in
   `src/app/components/fleet-detail-modal/fleet-detail-modal.ts` that sends unit
   telemetry to the model.

## Notes for a Claude Code session (vs. the original Gemini CLI)

- `codelab.md` and `GEMINI.md` were written for **Gemini CLI**. The *what* (files,
  goals, prompts) is tool-agnostic and still applies; the *setup* parts
  (`gemini-sdk` / `gemini-interactions-api` skills, `.gemini/settings.json`,
  `/skills` `/mcp`) are Gemini-CLI-specific — adapt them to your own tooling.
  `GEMINI.md` is Gemini CLI's agent-config file; this `CLAUDE.md` is the Claude
  equivalent.
- **Architecture choice:** the codelab wires Gemini **client-side** from
  `FleetService`/components. A cleaner alternative is a **server-side** endpoint in
  `src/server.ts` (Express) that calls Google's AI SDK and returns typed results —
  this keeps the API key off the browser and gives a typed server interface. Either
  is valid; pick deliberately and keep the key out of client code.

## Understand-Anything graph

A knowledge graph for this project lives in `.ua/` (committed separately). Explore
it with `/understand-anything:understand-dashboard logistics-manager-app`.
