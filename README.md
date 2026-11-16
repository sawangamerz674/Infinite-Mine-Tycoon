# Infinite Mine Tycoon

A 2D pixel-art incremental idle mining tycoon for the browser. Tap to mine, automate with machines and workers, dig through five regions, hunt treasure chests, and prestige for permanent bonuses. Built with plain HTML5, CSS3, vanilla ES modules and Canvas 2D: no framework, no build step, no runtime dependencies.

## Features

- 9 minerals across 5 themed regions, with rare discoveries and glowing gems
- 10 machines (producers and support), 5 worker types, 8 upgrade categories
- 20 research technologies in 5 categories, 22 achievements
- Prestige with Ancient Cores and a 7-upgrade permanent tree
- Automatic selling when storage is full (built in, switchable), offline progress
- Treasure rooms, milestone celebrations, particle effects, procedural music and sound
- Versioned, checksummed saves with backup, export and import
- Optional AI "Foreman" tips from Google Gemini through a small key-holding server (hidden when unavailable)
- CrazyGames SDK v3 integration (optional ads and cloud saves); installable as a PWA

## Quick start

Requires Node 22 or newer for the helper scripts and the advisor server (the game itself needs only a static web server).

```bash
npm start            # http://localhost:8080
```

Any static server works, for example `python3 -m http.server 8080`. Opening `index.html` straight from disk does not work because ES modules and `fetch` need `http://`.

## Scripts

| Command | Purpose |
|---------|---------|
| `npm start` | Static dev server (`tools/serve.mjs`) |
| `npm run start:ai` | Game plus the advisor API (needs `GEMINI_API_KEY`); `start:ai:mock` for a keyless demo |
| `npm run lint` | ESLint |
| `npm run validate` | Checks the JSON content files (ids, numbers, cross references) |
| `npm test` | Unit tests |
| `npm run test:e2e` | Browser smoke tests (needs Playwright) |
| `npm run check` | Lint, validate and unit tests |

## Project structure

```
index.html            Page shell and title screen
manifest.webmanifest  PWA manifest        sw.js  Service worker
config/               Tunable settings (game, balance)
data/                 Content: minerals, machines, workers, upgrades, regions,
                      research, achievements, prestige
css/                  Styles split by concern (main, layout, game, panels, animations, responsive)
js/
  main.js             Boot sequence
  core/               Game, GameLoop, GameState, EventBus
  systems/            Game rules (economy, production, workers, saves, audio, ...)
  rendering/          Canvas scene, sprites, icons, effects, animation, asset cache
  ui/                 DOM panels, HUD, modals, toasts
  integrations/       CrazyGames SDK wrapper
  data/               DataManager (loads and validates JSON)
  utils/              Pure helpers (math, number formatting, validation, time)
assets/               Icons, fonts, promotional images
docs/                 Architecture, game design, store and video guides
tests/                Unit and end-to-end tests
server/                Optional advisor API (Node, holds the Gemini key)
tools/                Dev server and data validator
```

## Configuration

Gameplay numbers live in JSON, never in code:

- `config/game.config.json`: virtual canvas size, offline limits, save interval, ads, treasure chests, milestones, asset overrides
- `config/balance.config.json`: storage, machine milestones, selling and research values
- `data/*.json`: every mineral, machine, worker, upgrade, region, technology, achievement and prestige upgrade

Run `npm run validate` after editing data. See [docs/GAME_DESIGN.md](docs/GAME_DESIGN.md) for how the numbers are used.

## Documentation

- [Architecture](docs/ARCHITECTURE.md): modules, data flow, events, saves
- [Game design](docs/GAME_DESIGN.md): rules, formulas and tuning
- [AI advisor](docs/AI_ADVISOR.md): setup, safeguards, platform notes
- [CrazyGames submission](docs/CRAZYGAMES_SUBMISSION.md)
- [Google Play release](docs/PLAYSTORE.md)
- [YouTube kit](docs/YOUTUBE.md)
- [Changelog](CHANGELOG.md)

## Debugging

`window.game` is exposed on `localhost` and when the URL contains `?debug`, which the tests and console experiments use. It is not available on production domains.

## Credits and licences

All sprites, icons, music and sound effects are generated in code. The bundled fonts (Pixelify Sans, Nunito) are under the SIL Open Font License; the licence texts are in `assets/fonts/`.
