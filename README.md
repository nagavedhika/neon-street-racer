# Neon Street Racer

A polished 2D top-down neon arcade highway racer built with Python and
Pygame. Dodge traffic, grab power-ups, burn nitro, and chase a high
score that's saved locally between runs. All visuals and sounds are
generated procedurally in code — no external image or audio files are
required, and the game needs no internet access to run.

## Description

You drive a neon sports car up a vertically scrolling four-lane
highway. Traffic spawns ahead of you and closes in as you drive;
weave between lanes to avoid collisions, collect power-ups, and burn
nitro for bursts of speed. The longer you survive and the further you
drive, the higher your score climbs — and the harder the traffic gets.

## Features

- Smooth, inertia-based car physics (acceleration, braking, steering)
- Four-lane scrolling highway with animated lane markings and glowing
  road edges
- Enemy traffic with random lane selection, varied speeds, and
  fairness checks so spawns never trap the player unfairly
- Progressive difficulty: traffic gets faster and denser over time
- Collision system with brief invulnerability and visual/audio feedback
- Three power-ups: **Boost**, **Shield**, and **Repair**
- Nitro boost system with a regenerating meter (`SPACE`)
- Distance-based scoring with a bonus for passing enemy cars
- Locally persisted high score (`data/highscore.json`)
- Full menu flow: Main Menu, Instructions, Pause, and Game Over screens
- Procedurally generated neon visuals (cars, glow effects, particles,
  speed lines, screen shake) and procedurally synthesized sound
  effects — nothing to download, nothing copyrighted
- Runs safely even with no audio device available (silent fallback)
- Window can be resized; the game scales to fit while keeping its
  aspect ratio

## Controls

| Key                  | Action                        |
|-----------------------|-------------------------------|
| `W` / `Up Arrow`       | Accelerate                    |
| `S` / `Down Arrow`     | Brake / Reverse               |
| `A` / `Left Arrow`     | Steer left                    |
| `D` / `Right Arrow`    | Steer right                   |
| `SPACE`                | Nitro boost                   |
| `ESC`                  | Pause / return from menus     |
| `R`                    | Quick-restart after Game Over |
| `Up` / `Down` + `Enter`| Navigate menus                |

## Requirements

- Python 3.8 or newer
- `pygame` (see `requirements.txt`)
- A Linux desktop environment with a graphical display (X11 or
  compatible). Kali Linux is fully supported.

## Installation

```bash
sudo apt update
sudo apt install python3 python3-pip

# From the project root:
python3 -m pip install -r requirements.txt
```

If your distribution's Python is externally managed and pip refuses a
system-wide install, use a virtual environment instead:

```bash
python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt
```

## How to Run on Kali Linux

```bash
cd neon-street-racer
python3 main.py
```

The game opens in a standard graphical window at 1280x720. If you
installed into a virtual environment, activate it first
(`source venv/bin/activate`) before running `main.py`.

## Running in a browser (localhost) and deploying to Vercel

This is still the exact same game/code — `main.py` and `src/game.py`
only had their main loop changed to `async`/`await asyncio.sleep(0)`
per frame, which is required by [pygbag](https://github.com/pygame-web/pygbag),
the tool that compiles Pygame to WebAssembly so it can run inside a
browser tab. That one change is a no-op on desktop (`python3 main.py`
still works exactly as documented above) and is what lets the same
source compile to run in a browser.

**Important:** Pygame itself is a native desktop framework (it opens a
real OS window via SDL). It cannot run on Vercel, or in any browser,
without a WebAssembly build — there is no way around that step. pygbag
is what produces that build.

### 1. Install pygbag (once)

```bash
pip install pygbag
```

### 2. Build + preview at localhost

From the project root:

```bash
python3 -m pygbag main.py
```

This does two things:
- Compiles the game to WebAssembly into `build/web/`
- Serves that folder at **http://localhost:8000** so you can play it
  in your browser immediately

The first build downloads pygbag's WASM runtime files from its CDN, so
it needs a normal internet connection the first time (results are
cached locally after that). Leave this running and open
`http://localhost:8000` in your browser to play.

### 3. Deploy to Vercel

This repo already ships a **ready-to-deploy `public/` folder** (the
compiled `index.html`, `neon-street-racer.apk`, `neon-street-racer.tar.gz`,
`favicon.png`) plus a `vercel.json` that points Vercel's output
directory at it and sets the correct `Content-Type`/cache headers for
the `.apk`/`.tar.gz` payloads — Vercel's default guess for those
extensions is wrong, and that mismatch is exactly what caused earlier
deploys to hang on the loading screen. With this `vercel.json` in
place, just:

```bash
npx vercel --prod
```

from the repo root, or import the repo in the Vercel dashboard
(framework preset **Other** — `vercel.json` handles the rest).

**If you change the game and need to rebuild `public/`:**

```bash
pip install pygbag
python3 -m pygbag --archive --build .   # writes build/web/
cp build/web/index.html build/web/favicon.png \
   build/web/*.apk build/web/*.tar.gz public/
```

Then commit and push the updated `public/` folder.

## Project Structure

```
neon-street-racer/
│
├── main.py                # Entry point
├── requirements.txt        # Python dependencies
├── README.md
├── .gitignore
│
├── assets/
│   ├── images/             # Reserved for future external art (unused —
│   │                         all visuals are drawn procedurally)
│   └── sounds/              # Reserved for future external audio (unused —
│                             all sound effects are synthesized in code)
│
├── data/
│   └── highscore.json       # Persisted local high score
│
└── src/
    ├── __init__.py
    ├── game.py              # Main game class, state machine, game loop
    ├── player.py             # Player car physics, nitro, power-up state
    ├── enemy.py               # Enemy traffic spawning and behavior
    ├── road.py                 # Scrolling road rendering
    ├── powerup.py               # Power-up spawning, pickup, rendering
    ├── particles.py              # Particle effects (speed lines, sparks, nitro flame)
    ├── hud.py                     # In-game HUD widgets
    ├── menu.py                     # Main menu, instructions, pause, game over screens
    ├── audio.py                     # Procedural sound effect synthesis and playback
    ├── highscore.py                  # High score load/save (JSON)
    └── settings.py                    # Central tunable constants
```

## Gameplay Explanation

You start in the main menu. Choose **Start Game** to begin driving.
Hold `W` to accelerate up to your top speed; use `A`/`D` to change
lanes. Enemy cars spawn at the top of the screen in random lanes and
approach at varying speeds — steer around them. Colliding costs one of
your three lives and grants a short window of invulnerability (your
car flashes) so you have a chance to recover. Power-ups drift down the
road in random lanes:

- **Boost** (amber lightning icon) — temporarily raises your top speed.
- **Shield** (cyan shield icon) — temporary immunity to collisions.
- **Repair** (green plus icon) — restores one lost life (up to the max
  of 3).

Nitro is separate from power-ups: hold `SPACE` any time to spend your
nitro meter for a strong, temporary speed multiplier. The meter
depletes while active and slowly regenerates when idle.

Your score increases continuously with distance traveled, plus a bonus
each time you pass an enemy car. Difficulty ramps up gradually over
about the first minute and a half of play — spawn rate increases and
enemy speeds rise — so early driving is forgiving while later driving
demands sharper reflexes. Losing all three lives ends the run and
shows the Game Over screen with your final score, high score, and
distance, plus options to restart, return to the main menu, or quit.

## Troubleshooting

**The window doesn't open / `pygame.error: video system not initialized`**
Make sure you're running in a graphical desktop session with a display
available (`echo $DISPLAY` should print something like `:0` or
`:1`). Headless servers and pure SSH sessions without X forwarding
cannot show a graphical window. If you're in a container, X11 must be
forwarded to the host display (see the note below about future
Dockerization).

**`ModuleNotFoundError: No module named 'pygame'`**
Run `python3 -m pip install -r requirements.txt` from inside the
project root, and make sure you're using the same Python environment
you used to install it (check `which python3` and, if using a virtual
environment, that it's activated).

**No sound effects play**
The game synthesizes its sound effects at startup using pygame's audio
mixer. If no audio device is available (common on some VMs or minimal
containers), the game automatically falls back to silent mode and
continues to run normally — this is expected behavior, not a bug.

**The game feels slow or choppy**
Confirm your VM/session has hardware-accelerated graphics available if
possible, and close other GPU/CPU-heavy applications. The game targets
60 FPS but will still run correctly, just less smoothly, on slower
hardware.

**High score doesn't save**
Ensure the `data/` directory is writable by your user account. The
game creates `data/highscore.json` automatically and fails gracefully
(falling back to an in-memory-only high score for that session) if it
cannot write to disk.

## Future Improvements

- Additional car skins/color choices selectable from the main menu
- More power-up variety (e.g. score multiplier, slow-motion)
- Online/shared leaderboard support
- Configurable key bindings
- Additional road environments (night city, desert, tunnel)
- Controller/gamepad support

## License

This project is provided as-is for personal, educational, and
portfolio use. All code and procedurally generated assets are original
and free of third-party copyrighted material.
