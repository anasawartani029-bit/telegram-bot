# Workspace

## Overview

pnpm workspace monorepo using TypeScript. Each package manages its own dependencies.

## Stack

- **Monorepo tool**: pnpm workspaces
- **Node.js version**: 24
- **Package manager**: pnpm
- **TypeScript version**: 5.9
- **API framework**: Express 5
- **Database**: PostgreSQL + Drizzle ORM
- **Validation**: Zod (`zod/v4`), `drizzle-zod`
- **API codegen**: Orval (from OpenAPI spec)
- **Build**: esbuild (CJS bundle)

## Key Commands

- `pnpm run typecheck` — full typecheck across all packages
- `pnpm run build` — typecheck + build all packages
- `pnpm --filter @workspace/api-spec run codegen` — regenerate API hooks and Zod schemas from OpenAPI spec
- `pnpm --filter @workspace/db run push` — push DB schema changes (dev only)
- `pnpm --filter @workspace/api-server run dev` — run API server locally

## First4Fitness Telegram Bot

Lives inside `artifacts/api-server`. Starts automatically next to the Express server.

- Entry: `artifacts/api-server/src/bot.ts` (launched from `src/index.ts`)
- Library: `telegraf` (long-polling)
- Required secret: `BOT_TOKEN` (Telegram bot token)
- Local data files:
  - `artifacts/api-server/assets/questions.json` — quiz bank: ~120-140 Arabic questions per topic (~390 total). Schema: `{question, options[4], answer (0-3 index), explanation}`.
  - `artifacts/api-server/assets/logo.jpg` — branding logo
  - `artifacts/api-server/assets/fonts/Amiri-Regular.ttf` & `Amiri-Bold.ttf` — Arabic font for PDF certificates (real TTFs from the Amiri 1.000 release)
- Topics: anatomy / functional / physiology — all questions in Arabic with English anatomical terms in parentheses
- Commands: `/start`, `/menu`, `/exam` (start exam), `/score`, `/leaderboard`
- Free-quiz mode: random questions per topic, retry on wrong first attempt, explanations after every answer. Per-session no-repeat tracking — each user's shown questions per topic are remembered until `/start` (which resets the session). When a topic is exhausted, the bot shows "لقد أنهيت جميع أسئلة هذا المجال 🎉" and offers other topics.
- Question validation: each question is checked on load (4 options, valid answer index 0-3, non-empty text). Invalid questions are skipped with a `WARN` log instead of crashing.
- Comprehensive exam mode (الاختبار الشامل): the **only** path to a certificate. Triggered from the main menu button or `/exam`. Builds 30 mixed questions (10 random unique per topic, then shuffled together so the topics interleave). Single attempt per question, no retry, no per-question explanation, no topic switching, no repeats inside one exam. Final report shows correct/wrong/percentage. Pass threshold = **80%**: if reached, a PDF certificate is auto-generated, sent via Telegram, and deleted from disk; otherwise the user sees "❌ لم تحقق نسبة النجاح المطلوبة للحصول على الشهادة". Safety: if a topic has fewer than 10 valid questions, the exam uses what is available rather than crashing.
- Certificate (`src/certificate.ts`): PDFKit + Amiri font, A4 landscape, includes logo, "شهادة إتمام" title, student name pulled from Telegram first/last name (falls back to `@username`, then `طالب <id>`), Arabic topic, score % and (correct/total), date (DD/MM/YYYY). Latin text and digits are rendered on their own lines to avoid PDFKit BiDi reversal. Safe fallbacks if logo/fonts/score data are missing.
- Persistent global leaderboard: `artifacts/api-server/data/leaderboard.json` (atomic writes, in-memory cache, recovers from missing/corrupt file).
- Per-user free-quiz session score is in-memory (resets on restart). The global leaderboard and certificate text are local — no external APIs, no DB, no scraping.

See the `pnpm-workspace` skill for workspace structure, TypeScript setup, and package details.
