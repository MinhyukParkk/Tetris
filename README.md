# Mono Blocks

A black-and-white falling-block puzzle game that runs from a single HTML file. There is nothing to install and no build step. Open the file in a browser and play.

Pieces are told apart by small marks instead of colors. The board follows your system's light or dark mode, and you can play with a keyboard or a touchscreen.

## Play it

1. Download `monoblocks.html` from this repository, or clone it with `git clone https://github.com/MinhyukParkk/Tetris.git`.
2. Double-click the file, or drag it into a browser window.
3. Select **Start game** or press **Enter**.

The game makes one network request, for its fonts. Offline, it runs the same with your system fonts.

To serve it locally instead, run this in the folder that holds the file, then open <http://localhost:8000/monoblocks.html>:

```sh
python3 -m http.server 8000
```

## Controls

### Keyboard

| Key | Action |
| --- | --- |
| ← → | Move |
| ↓ | Soft drop |
| Space | Hard drop |
| ↑ or X | Rotate clockwise |
| Z | Rotate counterclockwise |
| C or Shift | Hold |
| P or Esc | Pause or resume |
| Enter | Choose the main button in a dialog (Start game, Resume, Play again) |

Keys are read by physical key (`event.code`), so the controls keep working while a non-Latin input method, such as Korean, is turned on.

### Touch

On touchscreens and narrow windows, a button pad appears under the board with Hold, Soft drop, Hard drop, Rotate counterclockwise, Move left, Move right, and Rotate clockwise. Holding Move left or Move right repeats the move, and holding Soft drop keeps the piece falling fast.

## Features

- 10 × 20 board with two hidden rows above it
- 7-bag randomizer: each run of seven pieces contains every piece once
- Super Rotation System (SRS) rotation with wall kicks, including the separate I-piece kick table
- Hold (once per piece), a ghost piece that shows where the piece will land, and a queue of up to five next pieces
- Holding left or right waits 160 ms, then repeats every 45 ms
- 0.5 s lock delay that restarts when you move or rotate on the ground, up to 15 times; the count starts over each time the piece reaches a lower row
- T-Spin and T-Spin Mini detection, back-to-back bonus, combos, and All Clear bonus
- Best score saved in the browser
- Pauses automatically when you switch tabs or the window loses focus, and asks before **New game** ends a game in progress
- Follows the system's light or dark mode and reduced-motion setting
- Board and previews resize to fit the window and stay crisp on high-density screens

### Telling the pieces apart

| Piece | Mark |
| --- | --- |
| I | Solid, no mark |
| O | Small square |
| T | Dot |
| S | Diagonal line `/` |
| Z | Diagonal line `\` |
| J | Horizontal line |
| L | Vertical line |

In dark mode the tiles are white and the marks are black.

## Scoring

Line clears and T-Spins are multiplied by the current level.

| Clear | Points × level |
| --- | ---: |
| Single | 100 |
| Double | 300 |
| Triple | 500 |
| Quad (4 lines) | 800 |
| T-Spin, no lines | 400 |
| T-Spin Single | 800 |
| T-Spin Double | 1,200 |
| T-Spin Triple | 1,600 |
| T-Spin Mini, no lines | 100 |
| T-Spin Mini Single | 200 |
| T-Spin Mini Double | 400 |

Bonuses:

- **Back-to-back:** a Quad or a line-clearing T-Spin that follows another one scores 1.5 times its points. A Single, Double, or Triple in between breaks the chain. Pieces that clear nothing don't.
- **Combo:** from the second line-clearing piece in a row, each one adds 50 × combo count × level.
- **All Clear:** emptying the board adds 800, 1,200, 1,800, or 2,000 × level for a 1-, 2-, 3-, or 4-line clear.
- **Drops:** soft drop scores 1 point per row and hard drop scores 2 points per row.

A T-Spin counts when a T piece locks right after a rotation with at least three of the four cells diagonal to its center filled. Walls and the floor count as filled. It's a full T-Spin when both of those cells on the side the T points toward are filled, or when the rotation needed the last wall-kick test. Otherwise it's a Mini.

### Levels and speed

You start at level 1 and go up a level every 10 lines. At level 1 a piece falls one row per second. At higher levels it falls one row every `(0.8 - 0.007 * (level - 1)) ** (level - 1)` seconds, and the speed stops increasing after level 20. Soft drop is 20 times faster than the current speed.

The game ends when a new piece has no room to appear, or when a piece locks entirely above the visible board.

## How the code is organized

Everything is in `monoblocks.html`: the markup, one `<style>` block, and one `<script>`. The script is split into sections marked with comments:

| Section | What it holds |
| --- | --- |
| `rules` | Board size, piece shapes, SRS wall-kick tables, timing constants, and the fall-speed curve |
| `state` | Game state, the 7-bag, collision checks, moving and rotating, locking, T-Spin detection, scoring (`award`), hold, and game over |
| `elements` | References to the page's elements and canvases |
| `input` | Keyboard and touch handling, including auto-repeat for held keys |
| `game flow` | Starting, pausing, and resuming, the dialogs (`openOverlay`), and the clear callouts |
| `drawing` | Canvas rendering of the board, ghost piece, previews, and piece marks (`drawCell`) |
| `layout` | Sizing the board and previews to fit the window (`layout`) |
| `loop` | The per-frame update for gravity, lock delay, and animations, driven by `requestAnimationFrame` |
| `start` | The sample board behind the opening dialog, and startup |

### Common changes

| To change | Edit |
| --- | --- |
| Colors | The color variables at the top of the `<style>` block: `--paper`, `--ink`, `--ink-soft`, `--rule`, `--ghost`, and `--veil`. Light values are on `:root`, and the two dark-mode blocks below it hold the dark values. The canvas reads the same variables. |
| Fonts | The Google Fonts `<link>` and the `--font-display`, `--font-body`, and `--font-mono` variables |
| Handling | `DAS`, `ARR`, `LOCK_DELAY`, and `MAX_RESETS` in the `rules` section (times are in seconds) |
| Fall speed | `gravity(level)` in the `rules` section |
| Scoring | `award()` |
| Piece marks | `drawCell()` |
| On-screen text | The HTML between the `<style>` and `<script>` blocks, `openOverlay()`, and `CLEAR_NAMES` with `award()` for the clear callouts |

## Browser support

The game uses standard web APIs (Canvas 2D, Pointer Events, and `localStorage`) available in current versions of Chrome, Edge, Firefox, and Safari on desktop and mobile. If the browser blocks storage, as some private windows do, the game still runs but doesn't keep the best score.

## Fonts

[Anton](https://fonts.google.com/specimen/Anton) for headings, [Archivo](https://fonts.google.com/specimen/Archivo) for interface text, and [Martian Mono](https://fonts.google.com/specimen/Martian+Mono) for numbers and keys, all loaded from Google Fonts under the SIL Open Font License.