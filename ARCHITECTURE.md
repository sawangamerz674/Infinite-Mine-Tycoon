# Architecture

## Principles
- **Data-driven:** content and tuning live in `data/` and `config/`; code reads them through `DataManager`.
- **Decoupled:** systems never touch the DOM and the UI never owns game state. They talk through the `EventBus`.
- **Plain state:** `GameState` is a serialisable object; everything that must survive a reload lives there.
- **Safe numbers:** currency changes go through `GameState` helpers that reject NaN, Infinity and negatives.
- **No build step:** native ES modules served as-is.

## Boot sequence (`js/main.js`)
1. `CrazyGamesIntegration.init()` then `loadingStart()` (no-ops outside the portal)
2. Load `config/*.json` and `data/*.json` via `DataManager` (validated)
3. `new Game(config, data, platform)`, `platform.attach(game)`
4. `SaveSystem.load()`, register and preload sprites (`AssetLoader`), load fonts
5. `new UIManager(game)`, `loadingStop()`, title screen becomes interactive

## Module map

| Layer | Files | Responsibility |
|-------|-------|----------------|
| core | `Game`, `GameLoop`, `GameState`, `EventBus` | Composition root, fixed update loop with clamped delta, state, events |
| systems | `*System.js` | Rules: Mineral, Economy, Production, Mining, Machine, Worker, Upgrade, Research, Region, Achievement, Statistics, Prestige, Offline, Save, Settings, Audio, Boost, Treasure, Milestone |
| rendering | `CanvasRenderer`, `MineRenderer`, `SceneEffects`, `MachineVisuals`, `SpriteRenderer`, `IconFactory`, `ParticleSystem`, `AnimationManager`, `AssetLoader` | Everything drawn on the canvas; pure presentation |
| ui | `UIManager`, panels, `HUD`, `ModalManager`, `NotificationManager`, `StorageWidget`, `RewardedAds` | DOM panels that read state and call system methods |
| integrations | `CrazyGamesIntegration` | The only code that touches the SDK |
| server (optional) | `server/index.mjs`, `server/advisor.mjs` | Static files plus the advisor API; the only place the Gemini key exists |
| utils | `MathUtils`, `NumberFormatter`, `ValidationUtils`, `TimeUtils` | Pure, unit-tested helpers |

## Update order (per frame)
`GameLoop` calls `Game.update(dt)`: production, auto-sell (workers), mining boost, treasure, milestones, offline timestamp, autosave timer. `Game.render()` then draws the scene through `CanvasRenderer`. When the tab is hidden or a modal or ad is open, the loop is paused through `Game.setPaused(reason, on)`; several reasons can overlap.

## Events (`game.bus`)
Systems announce what happened and listeners react, for example `mined`, `sold`, `machinePurchased`, `upgradePurchased`, `regionUnlocked`, `mineralDiscovered`, `achievementUnlocked`, `treasureOpened`, `milestoneReached`, `prestiged`, `offlineEarnings`, `autoSellChanged`, `stateReplaced`, `modalOpened/modalClosed`. The full list with payloads is in the header comment of `js/core/Game.js`.

## Production and multipliers
`ProductionSystem` splits machine output across the region's minerals by spawn weight. Total output = machine levels x milestone bonuses x support machines x workers x research x achievements x prestige cores x boost. Additional multipliers register through `EconomySystem.registerMultiplier('production' | 'selling', fn)`.

## Rendering
The scene is drawn in a fixed 192x128 virtual space and scaled by a whole number, so pixels stay square. Region backgrounds, sprites and icons are generated once and cached by `AssetLoader`. Visual time is scaled by the player's animation-speed setting; gameplay timers are never scaled.

## Saves (`SaveSystem`)
- Envelope: `{ version, savedAt, checksum, data }` (FNV-1a checksum)
- Loading rebuilds state from known ids only, with finite non-negative numbers; unknown or hostile data is dropped
- A save is never written while game data is invalid; the previous good save is kept as a backup and unreadable saves are quarantined under a `_corrupt` key
- `MIGRATIONS` upgrades older versions step by step
- Export format `IMT1:` + base64; storage backend is the CrazyGames data module when active, otherwise LocalStorage

## Platform integration
`CrazyGamesIntegration` wraps documented SDK v3 calls only. Ads are optional; the game is fully playable without the SDK, with an ad blocker, or on any other domain.

## Testing
See [tests/README.md](../tests/README.md).

## Screen layouts
`css/responsive.css` defines one layout per screen class (no page scroll anywhere):
- Landscape desktop/tablet: scene left, side panel right, tab bar below.
- Portrait (phones, tablets, rotated windows) and narrow windows: scene on top, panel below.
- Landscape phones (height <= 480px): compact side-by-side layout and compact menu.

`js/ui/ScreenLayout.js` sets `--ui-zoom` (1x on phones, up to 2.2x portrait tablets, up to 2.5x desktop) so the HUD, panel, tabs, menus and dialogs stay readable on large screens; the scene is scaled by `CanvasRenderer`. iOS notch and home-indicator areas are respected through `viewport-fit=cover` and `env(safe-area-inset-*)`. `tests/e2e/layout.test.mjs` checks 21 screen sizes (run `npm start`, then `npm run test:e2e`).
