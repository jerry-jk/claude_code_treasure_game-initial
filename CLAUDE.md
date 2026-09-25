# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## About this project

React + Vite "Treasure Hunt" game, originally downloaded as a teaching example for learning Claude Code workflows. `README.md` in this directory is a workshop script of example prompts/commands (e.g. `/init`, `/clear`, `/context`, plan mode, custom slash commands) rather than project documentation — don't treat it as a spec.

## Commands

```bash
npm install
npm run dev      # Vite dev server on port 3000, opens browser automatically
npm run build    # outputs to build/
```

No lint or test tooling is configured — there are no `lint`/`test` scripts in `package.json` and no test files in the repo.

## Architecture

- Single-page React app; all game logic and rendering lives in `src/App.tsx`, mounted via `src/main.tsx`.
- Game state is 3 "boxes" (`useState<Box[]>`), one of which is randomly assigned `hasTreasure` in `initializeGame()`. Opening a box adjusts `score` (+100 treasure / -50 skeleton) and ends the game once the treasure is found or all boxes are opened.
- Animations use the `motion` package (Motion for React, i.e. Framer Motion's successor), imported as `motion/react` and used directly in JSX via `motion.div` props like `whileHover`/`animate`/`transition`.
- `src/components/ui/` is a full shadcn/ui component set (Radix UI primitives + `class-variance-authority` + Tailwind); most of these components are unused scaffolding from the template and only pulled in on demand — `App.tsx` currently only uses `Button`.
- Styling is Tailwind utility classes; global styles/tokens are in `src/index.css` and `src/styles/globals.css`.
- `vite.config.ts` aliases versioned import specifiers (e.g. `sonner@2.0.3`) to their unversioned package names — an artifact of the Figma-make/shadcn export this template originated from — and aliases `@` to `./src`.
- `src/guidelines/Guidelines.md` is an unfilled template for project-specific AI guidance; it currently has no real content.
