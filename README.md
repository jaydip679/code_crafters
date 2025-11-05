# Snake Game

A terminal-based Snake game built with C++ featuring cross-platform compatibility, thread-safe game logic, and persistent high score tracking.

---

## For Players

### Getting Started

Follow the install steps below for your OS, then run the compiled binary from a terminal in this folder. A `game_highest.txt` file will be created beside the executable to persist your best score.

### Game Screenshots

**Start Screen**
The welcome menu displays your high score and options to begin a new game or exit.
![Game Start Screen](https://github.com/maitry4/code_crafters/blob/main/Game%20Screenshots/game_start_screen.png)

**Initial Board Display**
The game board with instructions showing all control options before gameplay begins.
![Game Initial Board Display](https://github.com/maitry4/code_crafters/blob/main/Game%20Screenshots/game_initial_board_display.png)

**Gameplay in Action**
Real-time action showing the snake (displayed as `O` for head and `o` for body segments), food (`*`), and current score/length tracking at the top.

![Game Being Played](https://github.com/maitry4/code_crafters/blob/main/Game%20Screenshots/game_being_played.png)

**Game Over Screen**
Final score display with high score tracking and options to replay or quit. New high scores are highlighted.

![Game Over](https://github.com/maitry4/code_crafters/blob/main/Game%20Screenshots/game_over.png)

**Quit Confirmation**
Clean exit message displayed when you quit the game.

![Game Quit Message](https://github.com/maitry4/code_crafters/blob/main/Game%20Screenshots/game_quit_message.png)

#### Installation

**Windows:**
1. Install MinGW-w64 (or use MSYS2) with `g++` supporting C++20
2. Compile:
   - `g++ -std=c++20 main.cpp -o main.exe`
3. Run:
   - `main.exe` (or directly run this on command prompt (might get older version))

**Linux/macOS:**
1. Ensure you have `g++` installed
2. Compile:
   - `g++ -std=c++20 main.cpp -o snake_game`  
3. Run:
   - `./snake_game`

### How to Play

Navigate your snake around the board, eat the food (marked with `*`), and grow longer. Avoid hitting walls or yourself!

**Goal:** Eat as much food as possible to increase your score and beat your personal high score.

**Controls:**
- **Move Up:** `W` or `↑ Arrow Key`
- **Move Down:** `S` or `↓ Arrow Key`
- **Move Left:** `A` or `← Arrow Key`
- **Move Right:** `D` or `→ Arrow Key`
- **Quit:** `Q`

### Gameplay Rules

- Each food eaten grants **10 points**
- The snake grows by one segment for each food consumed
- The game ends if the snake:
  - Hits the boundary wall
  - Collides with itself
- After game over, press `R` to replay or `Q` to quit

### Features

- **High Score Tracking:** Your best score is automatically saved to `game_highest.txt`
- **Real-Time Score Display:** Monitor your current score, snake length, and high score at the top of the screen
- **Smooth Controls:** Responsive arrow key and WASD input handling
- **Cross-Platform:** Works seamlessly on Windows, Linux, and macOS

---

## For Contributors

### Project Overview

This is a terminal-based Snake game implementing sophisticated game design patterns: **lock-free thread-safe architecture**, **double-buffering for rendering**, and **platform abstraction layers**.

### Architecture

The codebase is organized into three main components:

#### 1. **Game Logic (`gameLogic.h`)**
The core game engine using **immutable state snapshots** and **atomic operations** for thread safety.

**Key Concepts:**
- **GameState Struct:** Immutable snapshot of the game at any moment (board state, snake position, score, etc.)
- **Lock-Free Threading:** Uses `std::atomic<shared_ptr<const GameState>>` to safely publish state from the game thread to the render thread without locks
- **Double Buffering:** Maintains `writeBuffer` and `readBuffer` to prevent torn reads during state updates
- **Direction Validation:** Prevents invalid moves (e.g., reversing 180° into yourself)

**Critical Methods:**
- `initializeBoard()`: Sets up the game with specified dimensions and starting configuration
- `update()`: Game loop tick—reads input, moves snake, checks collisions, publishes state
- `getGameState()`: Lock-free read of current game state (safe for render thread)

Additional details:
- State is published via atomic store with `memory_order_release` and read with `memory_order_acquire` to ensure visibility without locks.
- `initializeBoard()` seeds the RNG and places the initial snake centered with direction-based body placement.

#### 2. **Platform Abstraction (`main.cpp` - TerminalController)**
Handles all platform-specific terminal operations with unified API.

**Windows Support:**
- Uses `<conio.h>` for keyboard input (`_kbhit()`, `_getch()`)
- Uses Windows Console API for cursor positioning and screen clearing

**Linux/macOS Support:**
- Uses `termios` for raw input mode configuration
- Uses ANSI escape sequences for cursor control and screen manipulation
- Uses `fcntl` for non-blocking I/O

**Key Methods:**
- `clearScreen()`: Cross-platform screen clearing
- `setCursorPosition()`: Atomic cursor positioning
- `enableRawMode()` / `disableRawMode()`: Terminal configuration for Linux
- `kbhit()` / `getch()`: Cross-platform non-blocking keyboard input

#### 3. **Rendering & Input (`GameRenderer`, `InputHandler`)**
Handles visual output and user input processing.

**GameRenderer:**
- `drawFullScreen()`: Initial board layout with instructions
- `updateGameBoard()`: Efficient per-frame updates (only redraws game cells)
- `showGameOver()`: Game-over screen with score display

**InputHandler:**
- Handles arrow key sequences (different on Windows vs. Linux)
- Supports both arrow keys and WASD input
- `pollInput()`: Non-blocking input polling

Notes:
- Arrow keys on Windows use the `_getch()` extended key prefix; on POSIX, a short ESC sequence timeout is used for reliability.

### Tech Stack & Design Choices

| Technology | Rationale |
|---|---|
| **C++20** | Modern standard for atomic operations, smart pointers, and concurrency primitives |
| **std::atomic** | Lock-free thread safety without mutex overhead; minimal latency for input/render synchronization |
| **std::shared_ptr** | Automatic memory management for game state snapshots; safe concurrent access |
| **Double Buffering** | Prevents visual glitches (tearing) when updating display while game thread modifies state |
| **ANSI Escape Sequences** | Portable cursor control across Linux terminals; more reliable than system calls |
| **Terminal Raw Mode** | Non-blocking, responsive input without OS event loop overhead |

### Cross-Platform Compatibility

**Supported Platforms:**
- ✅ **Windows 7+** 
- ✅ **Linux** 

### Build & Run (Developer)

All source lives in the repo root for now:

```
.
├─ main.cpp          # Entry point: terminal UI, renderer, input, loop
└─ gameLogic.h       # Core game logic + immutable GameState snapshots
```

Commands:
- Windows (MinGW-w64):
  - `g++ -std=c++20 main.cpp -o main.exe`
  - Run with `main.exe`
- Linux/macOS:
  - `g++ -std=c++20 main.cpp -o snake_game`  
  - Run with `./snake_game`

Binary creates/reads `game_highest.txt` in the working directory for persistent high score.

### Contribution Guidelines

1. Fork and create a feature branch from `main`.
2. Keep edits focused; prefer small, cohesive changes.
3. Follow the code style and layout below.
4. Test on at least one platform (Windows or Linux/macOS). If you change terminal I/O, please sanity-test both.
5. Open a PR with a clear description, screenshots if UI/behavior changes, and build/run notes.

### Coding Style

- C++20, no exceptions unless necessary; avoid unnecessary try/catch.
- Use descriptive names; avoid cryptic abbreviations.
- Keep comments minimal and meaningful (non-obvious rationale, edge cases).
- Preserve existing indentation and formatting.

### Adding Features Safely

- New gameplay options (speed, board size, walls): expose config near `runGame()` in `main.cpp` and thread through to `initializeBoard()`.
- New cell types: extend `CellType` in `gameLogic.h`, update transitions in `update()`, and render mapping in `GameRenderer::updateGameBoard()`.
- Input changes: update `InputHandler::pollInput()` on both code paths (Windows and POSIX).
- Rendering changes: prefer buffering lines (as done) and a single flush per frame to avoid flicker.

