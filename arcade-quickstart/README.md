# Arcade: A Primer 
Arcade is a simple 2D game built with the Python Arcade library.

## Overview

This repository provides a ready-to-run copy of the game from the [RealPython Arcade Primer](https://realpython.com/arcade-python-game-framework/), with pinned dependencies and compatibility notes for modern Arcade versions. It’s intended as a fast-start reference and a compatibility guide for running the game on modern versions of the Python Arcade library.

This quickstart / compatibility guide is for anyone who wants to:
- **Run the Arcade Game immediately** — follow the quickstart for an out-of-the-box working version.
- **Modify or extend the game** — explore the code structure and use the [RealPython Arcade Primer tutorial](https://realpython.com/arcade-python-game-framework/) for detailed explanations of each part. 

## What this README Adds Beyond RealPython

- **Direct repo access** without needing to copy from the tutorial.
- **Pinned package versions** so the game runs as written.
- **Known Issues section** for modern Arcade releases.
- **Screenshots/GIFs** for quick visual reference.

Note: Instructions tested on macOS and Linux; Windows users may need small path adjustments. 

## Arcade Game Functionality 

- Sprite movement
- Soundtrack + sound effects
- Collision detection
- Score tracking 

## Prerequisites

**Important!** The game has not yet (as of August 2025) been updated to Arcade 3.0 or higher. It will only run on Arcade <3.0.

The game runs on Python version 3.7 or higher. However, if you are running Python 3.6, you can install a backport using pip:

```
python3 -m pip install dataclasses
```

## Dependencies

Arcade is a Python game framework built on top of several other libraries.  
When you install Arcade via `pip`, it automatically pulls in these dependencies:

Arcade (2.x)
- **pyglet** — windowing, graphics, audio, and input (built on OpenGL).  
- **numpy** — math ops for collisions and movement.  
- **Pillow** — loads/manipulates sprite images.  
- **pytiled-parser** — reads Tiled Map Editor files.  
- **shapely** (optional) — high-performance polygon math for collisions.  

### Dependency Details

- **`pyglet`** — Handles low-level windowing, graphics rendering, audio, and input.  
  - Relies on your operating system's graphics stack:  
    - **macOS:** Uses Cocoa APIs with OpenGL.  
    - **Windows/Linux:** Uses their respective OpenGL drivers.
- **`numpy`** — Optimized math operations, used in collision detection and sprite movement.
- **`Pillow`** — Image loading and manipulation (e.g., sprite textures).
- **`pytiled-parser`** — Reads map files created with the [Tiled Map Editor](https://www.mapeditor.org/).
- **`shapely`** *(optional)* — High-performance polygon math for complex collision detection.  
  - Note: Some versions may require workarounds on newer Python/macOS setups.

### System Requirements

Arcade needs:
- A working **OpenGL** environment.
- Compatible graphics drivers.
- On macOS: no headless mode; must run with an active display.
- On Linux: you may need additional system packages like `libgl1-mesa-dev`.

---

## QuickStart Instructions


```
git clone https://github.com/writeradam/arcade-a-primer/.git
cd arcade-a-primer
cd materials
python3 -m venv .venv
source .venv/bin/activate
pip install --upgrade pip
pip install "arcade<3.0"  # Arcade 2.x for tutorial compatibility
python3 arcade_game.py
```

## Quick Run (No Virtual Environment)

If you just want to try the game without creating a virtual environment, you can install Arcade system-wide and run it:

```
pip install --upgrade pip
pip install "arcade<3.0"
python3 arcade_game.py
```

⚠️ Note: This will install packages into your global Python environment, which may conflict with other projects. Use a virtual environment if you plan to keep developing or modifying the game.

## Arcade 2.x vs. 3.x Compatibility

The original RealPython code was written for Arcade 2.x. In Arcade 3.x, calling arcade.start_render() in a windowed game throws a RuntimeError:

```
RuntimeError: start_render() can only be called once during the application's lifetime...
```

**Why this happens:**

In Arcade 3.x, arcade.start_render() is no longer meant to be called in a typical on_draw() method for a game window. Instead, the rendering lifecycle is handled differently, and start_render() is reserved for specific one-shot drawing scenarios.

**How to fix it**
The easiest way to fix this issue is to install and run Arcade 2.x:

```
python3 -m pip install "arcade<3.0"
```

If you want to modernize the code, you can do so by updating every `arcade.start_render()` with a `selfclear()` or a `window.clear()` call.

## Running the Game

To launch the game, enter:

```
python3 arcade_game.py
```

A new window should pop up. Sound effects and gameplay should start immediately. If they do not, head to the next section of this tutorial.

<img width="1712" height="1340" alt="Screenshot 2025-08-14 at 3 03 46 PM" src="https://github.com/user-attachments/assets/0a65cded-6732-4ca2-9037-6c75befcc0a3" />

## PR Guidelines

1. Scope of Contribution

Only accept PRs that fix bugs, update compatibility for newer versions of Arcade, or improve documentation. No new gameplay features unless discussed in an issue first.

2. Branching Model

Create your feature branch from main and name it fix/<description> or feature/<description>. 

3. Commit Style

Use the [Conventional Commits](https://www.conventionalcommits.org/en/v1.0.0/) style: fix: `compatibility with Arcade 3.x.`

4. Testing Requirements

Make sure the game runs without errors using `arcade<3.0` and `arcade>=3.0`. Update README if fixing compatibility.

5. Documentation Requirements

If you fix a bug, update the “Known Issues” section of the README with the old behavior and the fix.

6. Review Process

PRs will be reviewed within 5 business days. Please respond to reviewer comments within a week or the PR may be closed.

## Type of license 

MIT

## Acknowledgments

Original game design, code, and tutorial content by the [RealPython](https://realpython.com) team. This repo’s purpose is quick setup and compatibility documentation.
