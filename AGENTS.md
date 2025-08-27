# Snakes And Letters - System & Rules Guide
This document defines the project structure and the game rules in detail. It is both a developer onboarding manual and a gameplay reference.

# Game Overview
- What it is: Snakes & Letters is a word-chain twist on Snakes & Ladders. Players advance on the board by forming valid words, with the word length controlling how far they move.
- Key twist: The last letter of the previous word (and its resolved board square) defines the required starting letter for the next word.
- Goal: Reach the final square first, navigating the board while managing vocabulary, wildcards, and strategy.
- Why it’s different: Unlike dice-only Snakes & Ladders, players must use language skills, tactical letter choices, and optional resist challenges against snakes.

# Game Loop Summary

At a high level, each game proceeds in this loop:
- Setup – Players seated, tokens placed on start square, rules/modes chosen, dictionary loaded.
- Turn Cycle – Roll → Word → Validate → Move → Resolve Snakes/Ladders → Pass turn.
- Continuation – Players alternate turns under the current ruleset.
- Win Condition – A player lands exactly (or as per mode rules) on the final square.
- Tie Breaks – If more than one player finishes in the same round, resolve by longest word, then fewest turns, then turn order.

This loop is fixed, but optional modes and variants adjust difficulty, timing, and penalties.

# Bots

- Purpose: Bots (agents) simulate players, allowing solo play or mixed play (human vs CPU).
- Design: Each bot follows a BotStrategy — a set of rules for selecting words given the current game snapshot.
- Current strategies:
  - RandomBot – picks any valid candidate word.
  - GreedyBot – prefers moves that land on ladders and avoid snakes.
  - TacticalBot – sabotages by ending on letters that make the next word harder.
- Fairness: Bots only use the same dictionary and rules as humans. Wildcards and rerolls apply equally.
- Future expansions: Difficulty levels (easy/medium/hard), human-like behavior (delays, mistakes), adaptive bots that learn player style.

# Dictionary System

- Role: The dictionary is the source of truth for word validation and candidate generation.
- Supported formats:
  - .txt (newline-separated words; simplest, good for MVP).
  - .json (structured words, categories, metadata).
  - Compressed DAWG/Trie (efficient lookup for large word sets).
  - API provider (server-side validation, useful in multiplayer).
- Word validation pipeline:
  - Check length = rolled requirement.
  - Check first letter = required last letter (unless wildcard).
  - Check no repeats if mode forbids.
  - Check presence in dictionary provider.
- Category dictionaries: Themed lists (animals, foods, places) enable Category Mode and educational variants.

# Modes & Variants

- Easy vs Challenge – Determines penalty for invalid word: stay put (Easy) or move backward by roll (Challenge).
- Timed Mode – Adds a countdown per turn (default 30s). Timeout = penalty or skipped turn.
- Category Mode – Restricts valid words to a chosen theme dictionary.
- Adaptive Mode – Alters dice weighting dynamically (easier words if struggling, harder if winning).
- Solo Puzzle Mode – Removes board; continue chaining words as long as possible. Useful for practice or training.
- Optional Rules:
  - No repeats vs limited repeats.
  - Overshoot handling (no move vs bounce).
  - Resist snake/ladder challenges (extra word difficulty).

# Directory Structure
```
/snakes-letters
├─ package.json              # Dependencies, scripts, metadata
├─ tsconfig.json             # TS compiler options (strict, paths)
├─ tsconfig.node.json        # Node/Vite specific TS config
├─ vite.config.ts            # Vite bundler setup
├─ index.html                # Entry HTML (mount point, manifest link)
├─ tailwind.config.cjs       # Tailwind config
├─ postcss.config.cjs        # PostCSS (tailwind+autoprefixer)
├─ .eslintrc.cjs             # ESLint rules
├─ .prettierrc               # Prettier formatting rules
├─ .gitignore
├─ README.md                 # Quickstart, build instructions
└─ public/
│  ├─ manifest.webmanifest   # PWA manifest
│  ├─ robots.txt             # SEO / crawler rules
│  ├─ icons/                 # PWA icons (192/512/maskable)
│  └─ assets/
│     ├─ sounds/             # roll.wav, move.wav, snake.wav…
│     ├─ images/             # token art, board background
│     └─ dictionary/         # english.txt (MVP), other langs
src/
├─ main.tsx                  # React entrypoint (mounts <App/>)
├─ app.tsx                   # App shell + Router
├─ styles.css                # Tailwind base + app-wide CSS
│
├─ pages/                    # Route-level components
│  ├─ Home.tsx               # Landing page, quick start
│  ├─ Game.tsx               # Main game screen
│  └─ Settings.tsx           # Rules toggles, theme, bots
│
├─ components/               # Reusable presentational UI
│  ├─ Board.tsx              # Renders board grid/SVG
│  ├─ Cell.tsx               # Single board cell
│  ├─ Token.tsx              # Player token sprite
│  ├─ Dice.tsx               # Dice UI + roll animation
│  ├─ WordInput.tsx          # Input bar + validation feedback
│  ├─ HUD.tsx                # Turn info, wildcards, timer
│  └─ Modals.tsx             # Help, settings, bonus challenge modals
│
├─ store/                    # Global game state (Zustand)
│  ├─ useGameStore.ts        # Central store (players, board, rules, actions)
│  ├─ selectors.ts           # Derived state helpers (e.g. canUseWildcard)
│  └─ actions.ts             # Mutations wrapping engine logic
│
├─ engine/                   # Pure game logic (framework-free)
│  ├─ rules.ts               # Turn flow, overshoot, snakes/ladders
│  ├─ dice.ts                # D6 → word length mapping
│  ├─ board.ts               # Index↔coords mapping, resolve snakes/ladders
│  ├─ validate.ts            # Word validation pipeline
│  ├─ types.ts               # Core types: Move, Phase, ValidationResult
│  └─ dictionary/
│     ├─ loader.ts           # Load + expose dictionary backend
│     ├─ textProvider.ts     # Loads .txt file
│     ├─ jsonProvider.ts     # Loads JSON
│     ├─ dawgProvider.ts     # DAWG/trie backend (future)
│     ├─ apiProvider.ts      # Server validation (future multiplayer)
│     └─ types.ts            # DictionaryProvider interface
│
├─ bots/                     # AI/Agent strategies
│  ├─ types.ts               # BotStrategy interface
│  ├─ randomBot.ts           # Baseline bot
│  ├─ greedyBot.ts           # Ladder-seeking bot
│  ├─ tacticalBot.ts         # Sabotage/tricky-letter bot
│  └─ index.ts               # Registry (id → strategy)
│
├─ hooks/                    # Game-related React hooks
│  ├─ useGame.ts             # Wrapper to access store state+actions
│  └─ useBotRunner.ts        # Monitors turn → invokes bot
│
├─ events/                   # Event buses (lightweight pub/sub)
│  ├─ audio.ts               # `audioEvents.emit('snake'|'ladder')`
│  └─ telemetry.ts           # Analytics events (optional)
│
├─ services/                 # Platform integrations
│  ├─ audio/
│  │  ├─ player.ts           # Centralized safe Audio() player
│  │  └─ map.ts              # Event → sound file mapping
│  ├─ storage.ts             # IndexedDB/localStorage wrapper
│  └─ telemetry.ts           # Analytics abstraction
│
├─ design/                   # Themes & styling tokens
│  ├─ tokens/
│  │  ├─ colors.light.json
│  │  └─ colors.dark.json
│  ├─ themes/
│  │  ├─ classic.theme.ts
│  │  └─ neon.theme.ts
│  └─ sprites/
│     ├─ snakes/…            # SVGs for snakes
│     └─ ladders/…           # SVGs for ladders
│
├─ config/                   # JSON configs
│  ├─ board.default.json     # 10x10 board + snakes/ladders mapping
│  └─ game.defaults.json     # Default rule toggles (wildcards, repeats…)
│
├─ i18n/                     # Translations (future expansion)
│  └─ en.json
│
├─ utils/                    # General helpers
│  ├─ random.ts              # RNG (optionally seeded)
│  ├─ result.ts              # Result/Option helpers
│  └─ viewport.ts            # Safe-area / keyboard handling
│
└─ sw.ts                     # Service Worker (PWA offline caching)

```

# Rules of Game

## Objective
- Reach the last square (e.g. index 99 on 10x10 board).
- Winner is first to finish.
- Tie-break: longest valid word in the winning round

## Setup
- Each player starts at square 0.
- Snakes drop you down, ladders raise you up (board defined in config/board.default.json).
- Default: English dictionary loaded (extendable).

## Turn Sequence

1. Roll the die
  - Maps to required word length.
  - Standard: 3–8 letters.
  - Advanced: occasional 9–10 letters (bonus challenge).
2. Form a word. The word must:
  - Match rolled length.
  - Exist in dictionary.
  - Start with the last letter of the previous word AND match the board position:
     - The square where the previous word ended defines the starting letter for the next word (or next player in case of multiplayer mode).
     - Example: If Player 1 ends on square “T”, Player 2 must form a word starting with “T”.
  - First turn: game picks random starting letter (fairness).
3. Validate
  - Fail reasons: wrong length, wrong start letter, repeated word, not in dictionary.
  - Wildcards: ignore start-letter requirement (2 per player by default). The last letter of the wildcard word still defines the next turn’s letter and square.
4. Move
  - Valid word → move forward = word length.
  - Invalid word →
    - Easy Mode: stay put.
    - Challenge Mode: move backwards by last die roll.
5. Snakes & Ladders
  - Ladder: climb immediately.
  - Snake: slide down unless you resist with a bonus word challenge (harder word).
  - Difficulty scales with snake/ladder size.
  - Bonus challenge: must be longer than rolled word (e.g. +1 for small snake, +2 for large). If impossible, resist fails.
  - Resolve chains repeatedly until stable (snake → ladder → …).
6. Pass turn
  - The last letter of your valid word becomes the required start letter for the next player.
  - This letter is bound to your final resolved square (after snakes/ladders).
   
## Edge Cases

- Impossible Word Situations
  - If no valid candidates exist for (length, required starting letter), reroll automatically.
  - Wildcards allow bypassing the letter/square constraint.
  - If no wildcards, concede a turn (Easy) or apply penalty (Challenge).
- Overshoot
  - Exact finish required.
  - If a move overshoots, player stays on current square (or “bounces” if that variant is chosen).
  - The last letter of the attempted valid word still defines the next required letter.
- Excessive Repetition
  - Prevent loops by disallowing word reuse.
  - Optional: max 2 uses per game.
- First Turn Fairness
  - Starting letter & square chosen randomly by game, not player.
- Snake/Ladder Balance
  - Resist mechanic prevents punishing loops.
- Sabotage Deadlock
  - Ending on rare letters (“Z”) may trap next player.
  - Wildcards + rerolls guarantee no deadlock.
- Word Validity
  - Allowed: A–Z, case-insensitive, accents normalized.
  - No hyphens/apostrophes.
  - No proper nouns.
  - No abbreviations.
  - Profanity blocklist enforced.
- Timeouts / AFK
  - In timed mode, failure = penalty or skip.
  - After multiple AFK turns, player may be auto-converted to bot.
- Ties
  - Multiple players finishing same round →
    - Longest word that round.
    - Fewest turns taken overall.
 
## Modes (To be selected at starting of the game) 
- Timed Mode → 30s per turn, fail = penalty.
- Category Mode → all words must fit a theme (animals, places, etc.).
- Adaptive Mode → dice weighting shifts depending on skill.
- Solo Puzzle → endless word chain, no board.

## Winning
- First to reach/past last square wins.
- If multiple reach in same round → tie-break rules above.

## Gameplay Summary

### Gameplay State Machine 
 [*] → Idle
  Idle → Rolling → Typing → Resolving → Animating → CheckFinish
   CheckFinish → GameOver
   CheckFinish → Idle (next player)
   Timeout (if timed) → Resolving (apply penalty)

### Turn Flow
 Start Turn
    ↓
 Roll d6 → requiredLength
    ↓
 Player submits word
    ↓
 ┌───────────────┐
 │ Validation OK?│
 └─────┬─────────┘
       │Yes
       ↓
   Advance = word length
       ↓
 Resolve Snakes/Ladders
 (repeat until stable)
       ↓
 Commit FINAL square
 + bind lastLetter → next-start
       ↓
 Finish? ──► Yes → GameOver
       │
       └──► No → Next player

### Bot Runner
Store: phase = Typing + seat = Bot
        ↓
 useBotRunner waits 600–1500ms
        ↓
 snapshot = selectSnapshot()
        ↓
 bot.pickWord(snapshot)
   │
   ├─ returns word → actions.submitWord(word,{fromBot:true})
   ├─ returns null + wildcard available → submitWord(any,{useWildcard:true})
   └─ else → nextPlayer()

### Letter–Square Coupling
Accepted Word → lastLetter
 lastLetter → sets next-start letter

 Movement = word.length
    ↓
 Resolve transitions (ladder/snake chain)
    ↓
 FINAL resolved square
    ↓
 Next-start letter is bound to FINAL resolved square
 (Overshoot = no move; keep current square, still set lastLetter)

# Design Philosophies
- Keep logic pure: All game mechanics (validation, movement, rules) must live in engine/ as pure functions with no UI dependencies. This ensures testability and predictable behavior.
- State as single source of truth: Store all mutable data (board, players, last letter, wildcards, mode) in store/. Components and bots consume only via selectors.
- UI as presentation only: Components display state and dispatch actions. They never embed rules or complex logic.
- Pluggable everything: Dictionaries, bots, themes, and rulesets should be swappable via interfaces, not hardcoded.
- PWA-first: Offline-ready, installable, and touch-friendly. App should feel native on mobile without extra builds.
- Accessibility matters: Clear contrast, ARIA announcements for dice and validation, minimum touch targets.
- Educational + Fun: Balance word-play skill with board randomness. Modes and themes should support both casual and educational use.
- Extensible: Future features (multiplayer, new bots, new dictionaries) should be add-ons, not rewrites.

# UI & UX

- Pages
  - Home:
    - Quick start game button.
    - Display last played game (resume).
    - Theme and mode selection.
  - Game:
    - Board (responsive grid or SVG).
    - Tokens for each player.
    - HUD (whose turn, required length, last letter, wildcards, timer).
    - Dice UI & roll animation.
    - Word input field with validation feedback.
    - Toasts/Modals for snakes, ladders, invalid words.
  - Settings:
    - Per-player controller (human or bot + bot type).
    - Rules toggles (timed, no repeats, wildcards count).
    - Theme chooser.
    - Dictionary provider selection.

- Components
  - Board & Cell: grid rendering with serpentine numbering.
  - Token: pawn sprites positioned on cells.
  - Dice: roll animation → maps to required length.
  - WordInput: input + wildcard toggle + feedback.
  - HUD: current player, last letter, word length, timer.
  - Modals: help, settings overlay, resist challenges.

- Design Goals
  - Responsive: Works on phones, tablets, desktops.
  - Minimal clutter: Players should see board + input without distraction.
  - Feedback-rich: Use animations, sound, and toasts to celebrate ladders, punish snakes, and confirm valid words.
  - Accessible: Use screen-reader-friendly announcements for dice results and required letters.
  - Skins/Themes: Board visuals, pawn styles, and sound packs are theme-driven.

# Bots
- Bots are modular strategies living in `src/bots/` .
- Each bot implements the BotStrategy interface and consumes a GameSnapshot.
- Built-in strategies:
  - RandomBot: baseline random choice.
  - GreedyBot: seeks ladders, avoids snakes.
  - TacticalBot: sabotages with tricky ending letters.
- Bot runner hook (`useBotRunner`) watches turns and invokes bot logic with a delay to simulate thinking.
- Future directions:
  - Difficulty tiers (easy → advanced heuristics).
  - Human-like bots (delays, occasional mistakes).
  - Adaptive bots that adjust to player style.

# Dictionary System
- Provider interface defines `has()` and `candidates()`.
- Supported providers:
  - Text provider (.txt, MVP).
  - JSON provider (.json).
  - DAWG/Trie provider (compressed, scalable).
  - API provider (server validation, anti-cheat in multiplayer).
- Category dictionaries: Support for theme-based play (animals, places, foods).
- Validation pipeline:
  - Length matches die roll.
  - Starting letter matches required last letter (unless wildcard).
  - Word not previously used (if mode forbids repeats).
  - Word exists in dictionary provider.
 
# Modes & Variants

- Easy Mode: invalid = no movement.
- Challenge Mode: invalid = move backward.
- Timed Mode: 30s countdown per turn.
- Category Mode: restrict dictionary to themed list.
- Adaptive Mode: dice weighting adapts to player success/failure.
- Solo Puzzle Mode: no board, continuous chaining challenge.
- Optional rules:
  - No repeats (or max 2 per game).
  - Overshoot handling (no move vs bounce).
  - Resist snake challenge (bonus harder word).

# Persistence & PWA

## Offline-First
- The app is a Progressive Web App (PWA) by design.
- App shell (HTML, CSS, JS bundles) is cached via a service worker for offline availability.
- Once a dictionary is loaded, it is cached for reuse offline.

## Local Storage
- Settings persistence:
  - Selected theme
  - Rule toggles (timed, no repeats, wildcards count)
  - Bot/human player assignments
- Saved games (optional):
  - Current board positions
  - Turn order, last letter, used words
  - Number of wildcards left

These can be stored in IndexedDB for structured snapshots or localStorage for lightweight flags.

## Installation & Add-to-Home-Screen

- When hosted on HTTPS and Lighthouse checks pass, the app prompts installation.
- PWA opens fullscreen with no browser chrome.
- Orientation locked to portrait by default.

## PWA Manifest

- Defined in `/public/manifest.webmanifest`.
- Contains app name, icons (192/512px, maskable).
- Defines theme color and background color for splash screen.

## Service Worker

- Cached assets: app shell, manifest, icons, and dictionary file(s).
- Updates automatically when new build is deployed.
- Fallback: if dictionary not cached offline, the app will block starting a new game until loaded.

# Themes & Design System

## Philosophy
- Visuals must be skinnable without altering core logic.
- All theme data flows through CSS variables and theme files in `src/design/`.
- Board graphics (snakes, ladders) and token sprites are stored per theme in `design/sprites/`.

## Structure
```
design/
├─ tokens/
│  ├─ colors.light.json
│  ├─ colors.dark.json
│  └─ spacing.json
├─ themes/
│  ├─ classic.theme.ts
│  ├─ neon.theme.ts
│  └─ kids.theme.ts
└─ sprites/
   ├─ snakes/
   │  ├─ classic/*.svg
   │  ├─ neon/*.svg
   │  └─ kids/*.svg
   └─ ladders/
      ├─ classic/*.svg
      ├─ neon/*.svg
      └─ kids/*.svg
```
## Theme Switching
- A `ThemeProvider` applies CSS variables to :root.
- Themes define:
  - Colors (board, HUD, dice, text)
  - Typography
  - Token shapes/colors
  - Snake/Ladder sprites
- Switching theme never alters game state.

## Accessibility

- Ensure high-contrast themes for visibility.
- Minimum 4.5:1 contrast ratio recommended.
- Provide a colorblind-friendly palette (future).

## Customization

- Developers can add new themes by:
  - Creating a `.theme.ts` file in `themes/`.
  - Adding assets in `sprites/`.
  - Registering theme in the theme registry.
- Players select theme in Settings.

# Testing & Quality

## Philosophy

- Pure functions must be tested. Everything in `src/engine/` should have unit tests.
- Store & actions are integration points. Test them to ensure game flow is correct.
- Bots need simulation tests. Confirm that strategies behave as described.
- UI tests should be minimal. Focus on state/logic; UI tests only cover key flows (input, dice, HUD updates).
- Reproducibility is key. Use seeded RNG in tests for deterministic results.

## Test Structure
```
/tests
├─ engine.rules.test.ts       # movement, snakes, ladders, overshoot
├─ engine.validate.test.ts    # word validation pipeline
├─ engine.board.test.ts       # index↔coords, serpentine mapping
├─ bots.greedyBot.test.ts     # prefers ladders, avoids snakes
├─ store.actions.test.ts      # rollDice, submitWord, nextPlayer
└─ integration.gameflow.test.ts
```

## What To Test
### 1. Engine
- rules.ts
  - Valid word moves forward by length.
  - Invalid word → stay (Easy) / move back (Challenge).
  - Overshoot handling (exact finish required).
  - Snake/Ladder resolution, including chaining.
  - Resist snake challenge scaling.

- validate.ts
  - Reject wrong length.
  - Reject wrong start letter unless wildcard.
  - Reject repeats if disallowed.
  - Reject absent words.
  - Accept valid word under all conditions.

- board.ts
  - Correct serpentine mapping.
  - Coordinates ↔ index round-trips.
  - Snake & ladder resolution order.

### 2. Store & Actions

- `rollDice()` sets requiredLength.
- `submitWord()` updates board + state correctly.
- Wildcards decrement correctly and still bind lastLetter.
- `nextPlayer()` cycles players in order.

### 3. Bots

- RandomBot always returns a candidate when available.
- GreedyBot chooses ladder over neutral move.
- TacticalBot prefers words ending with tricky letters (Q, Z, X).
- No infinite loops if no candidates exist (runner falls back to wildcard/skip).

### 4. Integration

- Full game simulation with seeded dictionary and RNG:
  - Ensure no deadlocks (rerolls + wildcards guarantee progress).
  - Ensure tie-breaking works (longest word > fewest turns > turn order).
  - Ensure lastLetter-square coupling is preserved across turns.

## Quality Gates

- Linting: ESLint with TypeScript + React hooks rules.
- Formatting: Prettier with project `.prettierrc`.
- Type safety: `tsconfig.json` with `strict: true`, `noUncheckedIndexedAccess`, and `exactOptionalPropertyTypes`.
- Accessibility audit: Run Lighthouse/axe for ARIA and contrast issues.
- PWA audit: Run Lighthouse to ensure installability and offline behavior.

## Continuous Testing (Optional)

- Add GitHub Actions workflow to:
  - Install deps.
  - Run `npm run lint`.
  - Run `npm run test`.
  - Run `npm run build` to confirm build integrity.

# Deployment

## Local Development
- Prerequisites:
  - Node.js ≥ 18 (LTS recommended).
  - Git (for version control).
- Install dependencies:
  ```
  npm install
  ```
- Run dev server:
  ```
  npm run dev
  ```
- Available scripts:
  - `npm run dev` → Start local dev server with hot reload.
  - `npm run build` → Create production build in `/dist`.
  - `npm run preview` → Serve production build locally for testing.
  - `npm run lint` → Run ESLint checks.
  - `npm run test` → Run Vitest unit tests
 
## Production Build
- Run:
  ```
  npm run build
  ```
- Outputs optimized app in `/dist` (minified, tree-shaken, ready for hosting).
- Contains `index.html`, JS/CSS bundles, manifest, service worker, and assets.

## Hosting Options
- Vercel (recommended)
  - Fast deploy, free tier, automatic HTTPS.
  - Steps:
    - Push repo to GitHub.
    - Import repo in Vercel
    - Framework preset: Vite.
    - Build command: `npm run build`.
    - Output directory: `dist/`.
    - Assign custom domain if desired.
- Netlify
  - Similar to Vercel, also free tier.
  - Steps:
    - Push repo to GitHub.
    - Import repo in Netlify
    - Build command: `npm run build`.
    - Publish directory: `dist/`.
- GitHub Pages
  - Works, but less smooth for PWAs due to routing issues.
  - Needs `vite.config.ts` base path adjustment if repo is not root.
- Cloudflare Pages
  - Simple, fast, global CDN.
  - Build command: `npm run build`.
  - Output directory: `dist/`.

## PWA Installability Checklist

- Manifest (`public/manifest.webmanifest`):
  - Name, short_name, icons (192px & 512px, maskable).
  - Theme color, background color.
  - Start URL = `/`.
  - Display = `standalone`.
- Service Worker (`src/sw.ts`):
  - Caches app shell.
  - Handles offline fallback.
- HTTPS required: Browsers only show install prompt over secure connection.
- Lighthouse audit:
  - Run Lighthouse in Chrome DevTools.
  - Ensure PWA installability passes.
  - Check performance, accessibility, best practices, SEO.

## Deployment Verification
- After deploying, test:
  - PWA prompt: Add-to-Home-Screen on mobile.
  - Offline mode: Disconnect and reload → game should still start.
  - Dictionary caching: Dictionary loads and validates words offline if previously cached.
  - Manifest & icons: Confirm splash screen, app icon, and fullscreen view.
  - Cross-device: Test on desktop + mobile (iOS/Android).

# Audio & Effects

## Philosophy
- Feedback-rich, not noisy: Sounds and animations should reinforce game events without overwhelming the player.
- Consistency: Each event type has a single, recognizable sound/animation.
- Optional: Players can mute or switch sound packs in Settings.
- Theme-driven: Visual effects (colors, sprites) and sound packs can change with the selected theme.

## Audio Events
The game uses a lightweight event bus (`audioEvents`) to trigger sounds.
- Events mapped to sounds:
  - `dice` → dice roll sound.
  - `move` → generic token move sound.
  - `ladder` → celebratory climb sound.
  - `snake` → falling/sliding sound.
  - `win` → fanfare sound.
  - `error` → invalid word / rejected action sound.

Sound files location:
```
public/assets/sounds/
├─ roll.wav
├─ move.wav
├─ ladder.wav
├─ snake.wav
├─ win.wav
└─ error.wav
```

- Playback behavior:
  - Sounds are preloaded on first user interaction (to satisfy browser autoplay policies).
  - Playback wrapped in `play().catch(() => {})` to gracefully ignore blocked attempts.

## Visual Effects
- Board & Tokens:
  - Token smoothly animates along squares when moving.
  - Ladders/snakes trigger short path animations (climb/fall).
- HUD Feedback:
  - Toasts or banners appear for events (e.g., “Snake resisted!”, “Invalid word”).
  - Word validation feedback appears inline in input box.
- Special Events:
  - Snake resist challenge: glowing token animation.
  - Ladder climb: highlight squares traveled.
  - Win: full-board confetti or theme-specific celebration.

## Theme Integration
- Themes can override:
  - Sound pack (e.g., classic board-game sounds, futuristic bleeps).
  - Animation styles (e.g., bouncy cartoony vs minimal sleek).
- Stored under `design/sprites/` (visual) and `public/assets/sounds/` (audio).

## Accessibility
- Mute toggle in HUD or Settings.
- Visual + audio redundancy: critical events (snake/ladder, invalid word) should always have a visual cue, not sound alone.
- Volume levels balanced; avoid clipping/distortion.

## Future Enhancements
- Haptic feedback on mobile when wrapped as app (dice roll vibration, snake drop shake).
- Voice pack support (e.g., text-to-speech reading out dice rolls and required letters).
- Custom user sound packs (upload or choose alternative sets).
