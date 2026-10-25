# Reflex — Reaction Timer

**Focus. Wait. React.**

Reflex is a single-file reaction timer built with vanilla JavaScript and Three.js. Wait for an animated amber shape to turn green and display **GO!**, then react as quickly as possible. Each valid attempt receives a time in milliseconds and a rating, alongside session statistics.

The project demonstrates browser timing, explicit game states, responsive interface design, accessible interaction, and real-time 3D rendering without a framework or build step.

## Features

- Centered welcome dialog with instructions and rating targets, shown once per page load.
- A fresh random wait of 1–5 seconds before each signal.
- Reaction measurement using `performance.now()`.
- Animated Three.js scene with a perspective camera, ambient and directional lighting, a point light, and a surrounding ring.
- Distinct shapes and colors for waiting, reacting, results, and false starts.
- Start, stop, retry, and session reset controls.
- False-start detection and a 15-second response timeout.
- Session best, average, valid attempt count, and the five latest results with individual ratings.
- Responsive layouts, keyboard controls, screen-reader announcements, and reduced-motion support.
- Playable CSS-based 2D fallback if the CDN or WebGL is unavailable.
- Embedded SVG favicon and all application HTML, CSS, and JavaScript in one file.
- In-memory results with no browser storage APIs or backend.

## Run locally

Save the game as `reaction-timer.html` and open it in a modern browser. No package installation or compilation is required.

For local development, you can also serve the containing folder with Python 3:

```sh
python -m http.server 8000 --bind 127.0.0.1
```

Then visit [http://127.0.0.1:8000/reaction-timer.html](http://127.0.0.1:8000/reaction-timer.html).

JavaScript is required. Internet access loads Three.js and the web fonts. If Three.js fails to load, the timer remains playable with a 2D signal; if the fonts are unavailable, system fonts are used. The file does not bundle its external dependencies for fully offline 3D use.

## How to play

1. Read the welcome guide and select **Got it, let’s play**.
2. Select **Start test**, activate the game area, or press **Space** while the game area or page is focused.
3. Wait while the amber shape displays **Wait for green…**.
4. When the signal becomes green and displays **GO!**, click or tap the game area, or press **Space**.
5. Review the time and rating, then start another round.

Reacting before the green signal produces a false start. False starts, stopped rounds, interrupted rounds, and timeouts do not affect the statistics.

| Control | Action |
| --- | --- |
| Click or tap the game area | Start a round or react to the signal |
| Space, with the page or game area focused | Start a round or react; held-key repeats are ignored |
| Enter, with a button focused | Activate that button |
| Escape | Close the welcome dialog or stop an active round |
| Stop | Cancel the current round without recording a result |
| Reset session | Cancel the round and clear all session results |

Standard button behavior applies when a separate control has keyboard focus. Resetting the session does not reopen the welcome dialog; reloading the page does.

## Score ratings

Lower scores are faster. Ratings use the displayed time, rounded to the nearest millisecond.

| Rating | Reaction time |
| --- | --- |
| Amazing | Under 180 ms |
| Very good | 180–219 ms |
| Good | 220–269 ms |
| Average | 270–349 ms |
| Below average | 350 ms or more |

These are game-defined targets, not validated population benchmarks. Display refresh, browser scheduling, device performance, and input method affect measurements. For consistent comparisons, use the same device and input method.

The average is calculated from the original, unrounded valid measurements and rounded for display. The best score and attempt count cover the entire current session, even though the recent-results list shows only five attempts.

## Technology and design

| Component | Implementation |
| --- | --- |
| Application | Semantic HTML, CSS, vanilla JavaScript |
| 3D library | Three.js `0.160.1`, dynamically imported as an ES module from jsDelivr |
| Timing and animation | `performance.now()` and `requestAnimationFrame()` |
| Resizing | `ResizeObserver` |
| Interface typography | Inter, with system font fallbacks |
| Heading typography | Outfit, with system font fallbacks |
| Favicon | Inline SVG data URL |

### Color palette

| Purpose | Color |
| --- | --- |
| Background | `#0F172A` |
| Surfaces | `#1E293B` |
| Primary text | `#F1F5F9` |
| Ready signal | `#10B981` |
| Waiting signal | `#F59E0B` |
| Results | `#3B82F6` |
| Errors | `#EF4444` |
| Secondary text | `#94A3B8` |

## Implementation notes

The game uses six explicit states: `idle`, `waiting`, `ready`, `result`, `false`, and `stopped`. State transitions update the visual stimulus, text, accessible labels, and enabled controls together.

The animation loop checks the random deadline. When it is reached, the game switches to green, submits the scene for rendering, and records `performance.now()`. The input handler captures another timestamp before updating the interface. Their difference becomes the reaction time. This measures browser-side elapsed time; it does not measure the exact instant pixels become visible on the physical display.

The waiting delay uses `crypto.getRandomValues()` when available and falls back to `Math.random()`. Late-loaded 3D graphics are activated between rounds so they do not replace the stimulus during an active test.

Geometry is allocated during scene initialization, and rendering pixel ratio is capped at `1.75`. The frame loop pauses while the document is hidden. Hiding the page or leaving the window cancels an active round, avoiding results inflated by interruptions.

When the page is unloaded, cleanup cancels animation work, clears the loading timeout and deferred initialization interval, aborts event listeners, disconnects the resize observer, and disposes of Three.js geometry, materials, and the renderer.

## Accessibility and resilience

- Native buttons and a modal dialog support keyboard interaction.
- Dialog dismissal returns focus to the game area.
- Visible focus indicators and live announcements communicate controls and results.
- Text and shape changes accompany color changes.
- `prefers-reduced-motion` disables rotation and floating motion.
- CDN failures, loading timeouts, rendering failures, and WebGL context loss switch the game to its 2D fallback.

Use a modern browser supporting native `<dialog>`, dynamic imports, `ResizeObserver`, and pointer events. WebGL is needed for the 3D presentation, but not for the fallback game.

## Data and privacy

Results live only in JavaScript variables. Reloading the page clears them. The application does not use cookies, local storage, session storage, IndexedDB, or a server to save scores.

Three.js is requested from jsDelivr, and fonts are requested from Google Fonts. These external requests still occur even though the game does not persist session data.

## Customization

Edit the single HTML file to change the experience:

- **Colors:** update the CSS variables in `:root` and the JavaScript `colors` map as needed.
- **Ratings:** update `scoreRating()` and both visible rating guides to keep the thresholds consistent.
- **Waiting range:** change `randomDelay()` and update the instructional text.
- **3D objects:** edit the `geometries` array inside `initGraphics()`.
- **Response timeout:** change the `15000` millisecond limit and its accompanying messages.
- **Branding:** edit the title, header, metadata, and SVG favicon data URL.

## Verification

The companion `verify-reaction-timer.cjs` script can be run from the project folder with Node.js:

```sh
node verify-reaction-timer.cjs
```

It checks game logic in a simulated browser environment, including timing boundaries, statistics, false starts, stop/reset, timeouts, input gating, rating boundaries, history labels, and module-loading failure handling. It also checks selected source-level layout and dependency rules. It does not render the page or verify real WebGL output.

For browser verification, check desktop and mobile layouts, keyboard navigation, welcome-dialog scrolling, reduced-motion behavior, all game states, and the fallback when the CDN is blocked.

## Deployment

Upload `reaction-timer.html` to a static web host. To serve the game at the root URL, publish a copy named `index.html`. No backend, environment variables, or build command is required. The Three.js and Google Fonts URLs must remain accessible for the full visual experience.