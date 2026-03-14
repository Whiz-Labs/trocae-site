# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Trocae-site is a static website built with **Astro 6** using the minimal template. TypeScript is configured in strict mode. No UI framework (React, Vue, etc.), CSS framework, testing, or linting is set up yet.

## Commands

- `npm run dev` — Start dev server (localhost:4321)
- `npm run build` — Production build to `./dist/`
- `npm run preview` — Preview production build locally

## Architecture

- **Astro file-based routing**: Pages in `src/pages/` map directly to routes
- **TypeScript**: Strict mode via `astro/tsconfigs/strict`
- **Node requirement**: >=22.12.0
