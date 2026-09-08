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

## Technologies Used

- **Python 3.11 / 3.8+** — Core game programming language
- **Pygame** — Game loop, window management, event handling, 2D graphics rendering, and procedural audio synthesis
- **Docker** — Containerized environment with SDL2 and X11 display dependencies

## Installation & Running Locally

### Prerequisites
- Python 3.8 or newer
- `pygame` (see `requirements.txt`)
- A graphical desktop environment (X11 / Wayland / Windows / macOS)

### Local Setup

```bash
# Clone the repository
git clone https://github.com/nagavedhika/neon-street-racer.git
cd neon-street-racer

# Install dependencies (or use a virtual environment)
python3 -m pip install -r requirements.txt

# Run the game
python3 main.py
```

If using a virtual environment:
```bash
python3 -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate
pip install -r requirements.txt
python3 main.py
```

## Running with Docker

### 1. Build the Docker Image
```bash
docker build -t neon-street-racer .
```

### 2. Run the Container
To run the graphical Pygame interface from Docker, allow local X11 display forwarding:

**On Linux (X11):**
```bash
xhost +local:docker
docker run -it --rm \
  -e DISPLAY=$DISPLAY \
  -v /tmp/.X11-unix:/tmp/.X11-unix \
  --device /dev/snd \
  neon-street-racer
```

**On Windows (using WSL2 / X server / VcXsrv):**
```bash
docker run -it --rm \
  -e DISPLAY=host.docker.internal:0 \
  neon-street-racer
```

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
