# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

topaê is a Portuguese-language (pt-BR) pre-launch marketing site for a João Pessoa-based item-swap app. Tagline: "Faz sua troca!". Two pages: a waitlist landing (`/`) and a 5-step survey funnel (`/pesquisa`). Built with **Astro 6** (minimal template), TypeScript strict, no UI framework. Node `>=22.12.0`.

The brand was previously "Trocaê" — the rebrand to "topaê" is complete in copy and visuals, but external references (Google Apps Script endpoint owner, Instagram handle `@trocaeoficial`) still point to the old name and will be migrated separately. Do not change those without confirmation.

## Commands

- `npm run dev` — dev server at `localhost:4321`
- `npm run build` — production build to `./dist/`
- `npm run preview` — preview built site
- `npm run astro check` — type-check `.astro` files (no test runner is configured)

## Architecture

**Two self-contained pages, no shared layout.** Each `.astro` file under `src/pages/` (and `src/components/PesquisaPage.astro`, which is just wrapped by `pages/pesquisa/index.astro`) carries its own full `<html>`, inline `<script>`, and `<style>` block. The brand CSS variables (`--laranja`, `--rosa`, `--turquesa`, `--verde`, `--pessego`, `--areia`, etc.) and `@font-face` declarations are **redefined per page** rather than imported — when changing palette or fonts, update both pages.

**Brand palette (from "Manual de Marca Topaê", source PDFs in `.claude/rebranding/`):**

| Token | Name | Hex | Use |
|---|---|---|---|
| `--laranja` | Laranja Goiaba | `#FE6520` | Primary CTA, brand wordmark, links |
| `--rosa` | Rosa Swipe | `#F53973` | Urgency/pulse accents, "swipe" cues |
| `--turquesa` | Azul Turquesa | `#22C4D9` | Secondary accent |
| `--verde` | Verde Profundo | `#1E8E78` | Success states, eco/positive accents |
| `--pessego` | Pêssego Pop | `#FDB94E` | Warning/info accents, badges |
| `--lilas` | Lilás | `#B68DEB` | Soft decorative |
| `--khaki`, `--oliva` | Khaki Fresh, Verde Oliva | `#CAD474`, `#92A263` | Nature accents |
| `--areia` | Areia Clara Digital | `#FDF4EB` | Page background |

Each color has paler tints (e.g. `--laranja-pale`, `--rosa-pale`) for backgrounds. The palette doesn't define a dark text color — `--text-dark: #2C2C2C` is a pragmatic fill, not from the brand spec.

**Fonts are self-hosted** as WOFF2 from `public/fonts/` (Aileron family — Regular/SemiBold/Bold/Heavy/Black — plus StoryScript-Regular). Each page declares its own `@font-face` block. Aileron is CC0 (sans-serif, used for everything); StoryScript is OFL (used for the hero tagline only). No Google Fonts CDN dependency. Source `.otf`/`.ttf` files live in `.claude/rebranding/` (Aileron in `aileron.zip`, StoryScript in `Story_Script.zip`) — if a new weight is needed, convert with `wawoff2` (`npm install --prefix /tmp/wawoff wawoff2`) and write the output to `public/fonts/`.

**Per-step accent colors in the survey.** `PesquisaPage.astro` sets a different `--step-color` and `--step-color-pale` on each `.step[data-step="N"]` (rosa → turquesa → pêssego → verde → laranja). The option cards then read those via `var(--step-color, ...)`, so each step's selected/hover styling auto-themes. The progress bar uses a 5-color gradient through the same sequence.

**Forms post to Google Apps Script, not a real backend.** Three hardcoded Apps Script URLs in the page scripts:

- Waitlist write — in `src/pages/index.astro` form handler
- Survey write — in `src/components/PesquisaPage.astro` form handler
- Signup count read (GET, returns `{count}`) — called on survey success; the UI adds `+20` to the returned count before displaying

All writes use `fetch(..., { mode: 'no-cors' })`, so the response is opaque and success is assumed whenever fetch doesn't throw. There is no real error path beyond network failure; do not add response-parsing logic for these endpoints.

**Survey referral tracking** comes from one of two sources, in order: the `?ref=` query param, then a `<meta name="surveyor-code">` populated at build time from the `PUBLIC_SURVEYOR_CODE` env var. The chosen value is sent as `codigo` in the survey payload.

**Survey flow** (`PesquisaPage.astro`): intro screen → 5-step form with auto-advance on radio/checkbox selection (q2 auto-advances when 2 boxes are checked; cap enforced by unchecking the offender). The Step 5 consent checkbox is opt-out: unchecking it reveals a FOMO warning but does not block submission — `consent` is sent as `sim`/`não` in the payload for the Apps Script to record.

**Assets in `public/`:**
- `logo.png` — Infinito symbol (orange infinity-with-arrows mark, used as favicon, nav icon, footer icon)
- `wordmark.png` — full "topaê" wordmark, used as the hero centerpiece
- `stickers/` — illustrated stickers ("Bora Trocar?", "Match Real", etc.) used decoratively on the landing hero
- `fonts/` — Aileron + StoryScript self-hosted

## Conventions

- All user-facing copy is Portuguese (pt-BR). Keep it that way.
- Brand spelling is **lowercase `topaê`** (with circumflex on the e). Not "Topaê", not "topae", not "Topae".
- Form submit handlers manipulate DOM display via inline `style.display` — match that pattern when adding states rather than introducing class-based show/hide.
- Inline SVG icons (lucide-style) are used everywhere; no icon library.
