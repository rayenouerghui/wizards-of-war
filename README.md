# project-c-game

[![Build Status](https://img.shields.io/badge/build-manual-yellow.svg)](https://github.com/rayenouerghui/project-c-game)
[![Version](https://img.shields.io/badge/version-0.1.0-blue.svg)]()
[![License: MIT](https://img.shields.io/badge/license-MIT-green.svg)](LICENSE)

A desktop 2D platformer/demo game written in C using SDL 1.2. This project is a compact, self-contained example of a classic 2D game architecture: menu system, animated player sprites, multiple backgrounds/levels, entities/pickups, and a simple score/health system.

Key features
- Written in C (C99-style), uses SDL 1.2, SDL_image, SDL_mixer and SDL_ttf
- Menu and options system (audio/music controls, background preview)
- Player selection screen and animated sprites (idle/walk/left/right)
- Multiple level backgrounds with loading animation and finish sequence
- Collectibles / entities and collision detection
- Score saving to `scores.txt` and simple minigames (enigma / tictactoe) triggered from gameplay
- Dockerfile provided to build and run in a contained environment

Screenshots
<!-- Use raw GitHub URLs for images so they render on GitHub -->
<p float="left">
  <img src="https://raw.githubusercontent.com/rayenouerghui/Wizards-of-War/main/menu%20game%20.png" alt="Menu screenshot" width="420" />
  <img src="https://raw.githubusercontent.com/rayenouerghui/Wizards-of-War/main/gameplay%20photo.png" alt="Gameplay screenshot" width="420" />
width="420" />
</p>

Quickstart (recommended for most users)
1. Install dependencies (Ubuntu/Debian example)
   - The game requires development libraries for SDL 1.2 and its helper libraries.
   - Example packages (Debian/Ubuntu):
     sudo apt-get update
     sudo apt-get install -y build-essential clang libsdl1.2-dev libsdl-image1.2-dev libsdl-mixer1.2-dev libsdl-ttf2.0-dev libsdl-gfx1.2-dev

2. Build & run with the included Makefile
   - The repository Makefile compiles and runs the game then cleans up the binary (it calls the program). To run:
     make prog
   - If you prefer to build without auto-running/cleaning (recommended for development), run the clang command shown in the Makefile:
     clang *.c -std=c99 -Ofast -lSDL -lSDL_image -lSDL_mixer -lSDL_ttf -lm -o prog
     ./prog

Run with Docker (isolated environment)
- The provided Dockerfile installs needed libraries, builds the game and sets SDL_AUDIODRIVER=dummy so audio won't fail inside containers.
- To run the native GUI game from Docker you must forward the host X11 display into the container:
  1. Allow local container access to your X server (run on the host):
     xhost +local:root
  2. Build the image:
     docker build -t project-c-game .
  3. Run the container and forward X11:
     docker run --rm -e DISPLAY=$DISPLAY -v /tmp/.X11-unix:/tmp/.X11-unix project-c-game
  4. (Optional) If you need sound in a container or different audio driver, adjust environment variables accordingly.

Headless testing
- To run the game without a display (e.g., automated tests or CI), use xvfb-run:
  xvfb-run -s "-screen 0 1366x768x24" ./prog
- Note: gameplay rendering won't be visible; this is useful for smoke tests.

Controls
- Movement:
  - Right arrow: move right
  - Left arrow: move left
  - Up arrow: jump (hold/release mechanics implemented in the code)
- Player selection screen: use the mouse to pick a player sprite
- Debug/other:
  - Space: (in code, decrements heart / can trigger tictactoe on death)
- No command-line options are required; run the compiled `prog` binary.

Project structure (high-level)
- .vscode/        - optional editor tasks/settings
- GAME/           - main game assets (backgrounds, sprites, sounds, fonts)
- MENU/           - menu assets and loading animations
- source/         - some additional source assets (icons, fonts, sounds)
- *.c, *.h        - C source and headers (main.c, game.c, menu.c, player.c, entity.c, background.c, utils.c, etc.)
- Makefile        - build + run target (compiles all .c files)
- dockerfile      - builds and runs inside Ubuntu container
- prog            - example compiled program (binary; can be removed or ignored)
- scores.txt, questions.txt - game data files

Notable source modules
- main.c — program entry; initializes subsystems and starts loops
- menu.c / menu.h — menu UI, options, loading screens, audio manager
- game.c / game.h — core game loop and state transitions (gameplay, minigames)
- player.c / player.h — player sprite loading, rendering, movement and jump physics
- entity.c / entity.h — pickups/entities, collision handling
- background.c / background.h — level/backdrops, loader and finish animation
- utils.c / utils.h — small helpers: surface loading, scaling, clamp

Development notes
- Language & style: the project targets C (compiled with clang in the Makefile) and uses -std=c99
- Libraries: SDL 1.2 + SDL_image, SDL_mixer, SDL_ttf. The Dockerfile lists exact packages used for Ubuntu 22.04.
- Adding/Editing assets:
  - Backgrounds and levels are stored under GAME/background/levelX; background loader code expects specific images and loading frames. To add a level you will need to update InitBackground() and levelloader() in background.c accordingly.
  - Player sprites are in GAME/perso/<player>/ with directories stand/, left/, right/ — keep naming/indices consistent with player.c loops.
- Save/score: scores are appended to `scores.txt` with saveScore() in background.c

Useful commands (copy-paste)
- Full local compile and run (single-line):
  clang *.c -std=c99 -Ofast -lSDL -lSDL_image -lSDL_mixer -lSDL_ttf -lm -o prog && ./prog
- Docker build & run (with X11)
  xhost +local:root
  docker build -t project-c-game .
  docker run --rm -e DISPLAY=$DISPLAY -v /tmp/.X11-unix:/tmp/.X11-unix project-c-game

Troubleshooting
- SDL_image/IMG_Load() returns NULL for missing images — ensure you run the binary from the repository root (paths are relative).
- If you see "Failed to initialize SDL" errors, verify that SDL and development libraries are installed.
- Fonts: TTF font files (ARCADECLASSIC.TTF, Pixeboy.ttf, etc.) are expected at project root and in source/. If TTF_OpenFont fails, the menu will print an error.
- If the Makefile appears to delete the binary immediately after running: the target `prog` in Makefile builds, runs and then removes `prog` and object files. Use the clang command above if you want to keep the binary for debugging.
- Running in Docker without forwarding X11: the game requires a display. Use xvfb-run for virtual framebuffer testing, or forward X11 as shown above.

Contributing
Thank you for your interest — contributions are welcome!
- Report issues: open a GitHub issue with a short description, steps to reproduce, and environment details (OS, SDL versions).
- Pull requests:
  - Fork and branch from main, use a descriptive branch name (e.g., fix/collision-bounds).
  - Keep C changes small and focused. Prefer clang-format style where reasonable.
  - If adding assets, include a short README for the asset source or license and place them under GAME/ or MENU/ with consistent naming.
  - Update code that assumes level counts (background.c) when adding new levels.
- Coding style:
  - Target C99, keep builds reproducible with the Makefile or clang command shown above.
  - Prefer descriptive variable names and keep platform-specific code isolated where possible.

Optional local convenience alias
- You can add a small alias to your shell config for quick run (example for bash ~/.bashrc or ~/.zshrc):
  alias run_game='clang *.c -std=c99 -Ofast -lSDL -lSDL_image -lSDL_mixer -lSDL_ttf -lm -o prog && ./prog && rm -f prog'
- Note: that alias builds, runs, and removes the binary (mirrors the Makefile). Adjust to keep the binary if you prefer.

License
This README recommends using the MIT license for portfolio projects. To apply it, add a `LICENSE` file with the MIT text below.

MIT License
Copyright (c) 2026 <Your Name>

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:
[...full MIT text truncated here in README — add a LICENSE file with the full text...]

(For completeness: add a proper LICENSE file at the repository root with the MIT full text.)

Acknowledgements
- Built using SDL 1.2 and associated libraries.
- Sprite and asset handling inspired by classic 2D platformers.

---

If you'd like, I can:
- generate a ready-to-add LICENSE file,
- produce a CONTRIBUTING.md template,
- or create an Issues/PR checklist to include in the repo.
