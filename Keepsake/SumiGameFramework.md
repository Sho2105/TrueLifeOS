# Sumi Framework

A minimal, modular, ultra-low-level 2D-first game framework written in C99.
No dependencies. No engine. No opinions. Just clean building blocks.

---

## Table of Contents

1. [Introduction](#introduction)
2. [Philosophy & Design Goals](#philosophy--design-goals)
3. [Core Architectural Principles](#core-architectural-principles)
4. [Module Overview](#module-overview)
5. [File & Directory Organization](#file--directory-organization)
6. [Naming Conventions](#naming-conventions)
7. [API Design Rules](#api-design-rules)
8. [Memory & Ownership](#memory--ownership)
9. [Threading Philosophy](#threading-philosophy)
10. [Platform Abstraction Philosophy](#platform-abstraction-philosophy)
11. [Rendering Philosophy](#rendering-philosophy)
12. [Error Handling Philosophy](#error-handling-philosophy)
13. [Build System Expectations](#build-system-expectations)
14. [Extension & Plugin Philosophy](#extension--plugin-philosophy)
15. [Complete Module Breakdown](#complete-module-breakdown)
16. [API Cheatsheet](#api-cheatsheet)
17. [Usage Examples](#usage-examples)
18. [Implementation Notes](#implementation-notes)

---

## Introduction

Sumi is a C99 framework that solves the hard, low-level problems of 2D game
development — windowing, rendering, input, audio, math, memory — so that
developers can build their own engine layer and games on top of a reliable,
consistent, well-understood foundation.

Sumi is not a game engine. It does not provide a scene graph, entity system,
scripting layer, editor, or asset pipeline. It provides building blocks.

Target users: developers who want full control over their game's architecture
but do not want to rewrite platform and graphics boilerplate for every project.

**Language:** C99 (C++ compatible)
**Dependencies:** None (platform libc only)
**Target platforms:** Windows, macOS, Linux (mobile as future extension)
**Primary backend:** OpenGL 3.3 Core (Metal/Vulkan as optional backends)

---

## Philosophy & Design Goals

### What Sumi Is

- A thin, uniform interface over platform, graphics, audio, and input
- A set of general-purpose data structure primitives
- A predictable, low-surprise foundation for building higher-level systems
- Modular: use only what you need, ignore the rest
- Self-contained: the framework compiles from source with no external dependencies

### What Sumi Is Not

- Not a game engine
- Not an ECS
- Not a scene graph
- Not a scripting runtime
- Not an asset pipeline
- Not a UI toolkit
- Not an HTTP client, networking library, or anything unrelated to making games

### Design Goals (In Priority Order)

1. **Correctness** — Defined behavior. No silent failures. No undefined state.
2. **Simplicity** — Small API surface. One obvious way to do each thing.
3. **Performance** — No hidden allocations. No virtual dispatch. Cache-friendly layouts.
4. **Portability** — Clean C99. Compiles on any target with minimal adaptation.
5. **Modularity** — Each module is independently usable. No forced coupling.
6. **Ergonomics** — Easy to read, easy to remember, easy to debug.

---

## Core Architectural Principles

### Principle 1: Explicit Over Implicit
No hidden state. No global context unless documented. Every function that reads
or mutates state takes that state as an explicit parameter.

### Principle 2: Flat Over Deep
No inheritance chains. No deep object hierarchies. Prefer flat structs and
free functions. Composition through containment, not inheritance.

### Principle 3: No Hidden Allocations
Every function that allocates memory either takes an allocator parameter or
documents exactly when and where it allocates. The caller is never surprised
by a malloc inside a seemingly-simple function.

### Principle 4: Data Is Dumb, Logic Is Separate
Structs hold data. Functions operate on structs. No methods. No vtables unless
explicitly implementing a plugin/backend interface.

### Principle 5: Opt-In Complexity
Simple use cases require simple setup. Advanced use cases unlock more control.
The defaults are good. The overrides are always available.

### Principle 6: Modules Don't Own Each Other
Modules communicate through well-defined output structs, not by calling into
each other's internals. A module may *use* types from another module but must
not reach into its internals.

### Principle 7: Single-Header, STB-Style Organization
Each module is a single `.h` file. Implementation is compiled in exactly one
translation unit by defining `SUMI_<MODULE>_IMPLEMENTATION` before including.
The amalgamation header compiles everything at once.

---

## Module Overview

| # | File | Responsibility |
|---|------|----------------|
| 1 | `sumi.h` | Amalgamation — includes all modules in correct order |
| 2 | `sumi_platform.h` | Window, OS context, GL context, clipboard, system time |
| 3 | `sumi_time.h` | Delta time, fixed timestep, frame counter, frame rate cap |
| 4 | `sumi_input.h` | Keyboard, mouse, gamepad, touch state queries |
| 5 | `sumi_mem.h` | Arena, pool, and scratch allocators |
| 6 | `sumi_math.h` | Vec2, Vec3, Vec4, IVec2, Rect, Mat3, Mat4, Transform2D, utils |
| 7 | `sumi_color.h` | Color types, conversion, palettes |
| 8 | `sumi_buf.h` | Dynamic array, ring buffer, string builder |
| 9 | `sumi_map.h` | Hash map and hash set |
| 10 | `sumi_event.h` | Event queue, event dispatch |
| 11 | `sumi_image.h` | PNG/BMP/QOI decode and encode (CPU only, no GPU) |
| 12 | `sumi_asset.h` | File I/O, virtual filesystem, asset loading and caching |
| 13 | `sumi_texture.h` | GPU texture creation, atlas, sub-texture, sampler state |
| 14 | `sumi_shader.h` | Shader compilation, uniform binding, built-in shaders |
| 15 | `sumi_gfx.h` | Sprite batch, draw calls, render targets, blend modes |
| 16 | `sumi_font.h` | Font atlas generation, glyph metrics, text draw calls |
| 17 | `sumi_camera.h` | Camera2D, world↔screen transforms, shake |
| 18 | `sumi_audio.h` | Audio device, sound playback, streaming, mixing |
| 19 | `sumi_collision.h` | AABB, circle, polygon tests, raycast, manifold |
| 20 | `sumi_debug.h` | Logging, assertions, debug draw overlay, profiling zones |

---

## File & Directory Organization

```
sumi/
├── sumi.h                  # Amalgamation header
├── sumi_platform.h
├── sumi_time.h
├── sumi_input.h
├── sumi_mem.h
├── sumi_math.h
├── sumi_color.h
├── sumi_buf.h
├── sumi_map.h
├── sumi_event.h
├── sumi_image.h
├── sumi_asset.h
├── sumi_texture.h
├── sumi_shader.h
├── sumi_gfx.h
├── sumi_font.h
├── sumi_camera.h
├── sumi_audio.h
├── sumi_collision.h
├── sumi_debug.h
├── examples/
│   ├── 01_window/
│   ├── 02_sprites/
│   ├── 03_input/
│   ├── 04_audio/
│   ├── 05_tilemap/
│   └── 06_full_game/
├── tests/
│   └── test_<module>.c     # Per-module unit tests
└── docs/
    └── <module>.md         # Extended per-module documentation
```

There is no `src/` directory. All implementation lives in the headers.
There is no `CMakeLists.txt` as the canonical build. See Build System Expectations.

---

## Naming Conventions

### Types

```c
SumiVec2        // Structs: Sumi + PascalCase
SumiRect
SumiTexture
SumiCamera2D
SumiKeyCode     // Enums: Sumi + PascalCase
SumiBlendMode
```

### Functions

```c
sumi_gfx_init()              // sumi_<module>_<verb>[_noun]()
sumi_gfx_draw_sprite()
sumi_gfx_set_blend_mode()
sumi_texture_create()
sumi_texture_destroy()
sumi_input_key_down()
sumi_input_mouse_pos()
```

### Constants & Enum Values

```c
SUMI_KEY_A                  // SUMI_<MODULE>_<NAME> or SUMI_<NAME>
SUMI_BLEND_ALPHA
SUMI_MAX_TEXTURES
SUMI_VERSION_MAJOR
```

### Internal (private to implementation)

```c
_sumi_gfx_flush_batch()     // Prefix with underscore
_sumi_platform_gl_init()
```

### Boolean Convention

```c
// Functions returning bool use present-tense verbs
sumi_input_key_down(key)     // true while held
sumi_input_key_pressed(key)  // true for one frame
sumi_input_key_released(key) // true for one frame
sumi_gfx_ready()
sumi_audio_is_playing(handle)
```

### Handles

Opaque integer handles for GPU resources and audio voices:
```c
typedef u32 SumiTextureID;
typedef u32 SumiSoundID;
typedef u32 SumiShaderID;
typedef u32 SumiRenderTargetID;
typedef u32 SumiFontID;
#define SUMI_INVALID_ID  0
```

Zero is always the invalid sentinel. A returned ID of 0 indicates failure.

### Primitive Typedefs

Defined once in `sumi_platform.h`, used everywhere:
```c
typedef uint8_t   u8;
typedef uint16_t  u16;
typedef uint32_t  u32;
typedef uint64_t  u64;
typedef int8_t    i8;
typedef int16_t   i16;
typedef int32_t   i32;
typedef int64_t   i64;
typedef float     f32;
typedef double    f64;
typedef uint8_t   b8;   // boolean
typedef uint32_t  b32;
```

---

## API Design Rules

1. **Init/Shutdown pairs** — Every subsystem that holds resources has
   `sumi_<module>_init()` and `sumi_<module>_shutdown()`. Calling them in
   wrong order is a programmer error.

2. **Create/Destroy pairs** — Resources (textures, shaders, sounds) are created
   and destroyed explicitly. No reference counting. No GC.

3. **No out-parameters for simple returns** — Prefer returning structs by value
   for small types (Vec2, Rect, Color). Use out-parameters only for large
   structs or when multiple returns are genuinely needed.

4. **Desc/Config structs for complex creation** — Any function with more than
   4 parameters uses a `Sumi*Desc` struct. This allows named fields and
   zero-initialization as defaults.
   ```c
   // Bad
   sumi_texture_create(data, w, h, fmt, filter, wrap_s, wrap_t, mips);

   // Good
   SumiTextureDesc desc = {
       .data   = pixels,
       .width  = 256,
       .height = 256,
       .format = SUMI_PIXEL_RGBA8,
       .filter = SUMI_FILTER_NEAREST,
   };
   SumiTextureID tex = sumi_texture_create(&desc);
   ```

5. **Zero-init is a valid default** — Desc structs must be designed so that
   `= {0}` produces sensible defaults wherever possible.

6. **Const correctness** — Input pointers that are not mutated are `const`.

7. **No varargs in hot paths** — Variadic functions are acceptable for debug
   logging only.

8. **No callbacks in hot paths** — Callbacks are acceptable for event dispatch
   and plugin interfaces. Never for per-frame rendering or update logic.

9. **No function pointers in public structs unless they are plugin interfaces**.

10. **Batching is the caller's job** — `sumi_gfx_draw_*` functions accumulate
    into the current batch. The caller calls `sumi_gfx_flush()` when ready.

---

## Memory & Ownership

### Rules

- Sumi does not own your memory unless you explicitly give it to Sumi.
- Functions that return a pointer to caller-owned memory are documented as such.
- Functions that return a pointer to internally-managed memory document its
  lifetime explicitly.
- Passing a pointer to Sumi means Sumi reads from it immediately and does not
  retain the pointer unless documented.

### Allocator Interface

Modules that allocate memory take an optional `SumiAllocator*`. If NULL is
passed, the default allocator (platform malloc/free) is used.

```c
typedef struct SumiAllocator {
    void* (*alloc)(usize size, void* ctx);
    void  (*free)(void* ptr, usize size, void* ctx);
    void* ctx;
} SumiAllocator;
```

### Arena Allocator (sumi_mem.h)

Used for frame-scoped or lifetime-scoped bulk allocation:
- Allocations are O(1), free is a single pointer reset
- No individual frees; entire arena is reset or destroyed
- Ideal for per-frame scratch work, string building, temporary arrays

### Pool Allocator (sumi_mem.h)

Used for fixed-size, frequently allocated/freed objects:
- Constant-time alloc and free
- No fragmentation
- Not thread-safe by default

### Ownership Convention Summary

| Pattern | Ownership |
|---------|-----------|
| `sumi_texture_create(&desc)` | Caller owns desc data; Sumi copies to GPU |
| `sumi_asset_read_file(path, &size)` | Returned pointer is caller's; caller frees |
| `sumi_font_load(data, size)` | Sumi copies font data internally |
| `sumi_image_decode(data, size, &img)` | SumiImage.pixels is heap-allocated; caller frees |
| `sumi_debug_log(msg)` | Sumi reads immediately; no retention |

---

## Threading Philosophy

Sumi is single-threaded by default. The entire API is designed to be called
from one thread (the main thread). There are no internal threads.

**Explicit threading for audio:** `sumi_audio.h` runs a background mixing
thread. The public API is thread-safe for `sumi_audio_play`, `sumi_audio_stop`,
and `sumi_audio_set_volume`. The mixing callback is internal and locked.

**No thread-safety guarantees elsewhere.** If you use threads, you are
responsible for synchronization when touching Sumi state from multiple threads.

**Recommended pattern for multi-threaded games:**
- Main thread: all Sumi API calls (input, rendering, audio triggers)
- Worker threads: game logic, AI, physics — work on your own data, then pass
  results to the main thread via your own queue before the next frame

Sumi does not provide a threading or job system. That is intentionally
out of scope.

---

## Platform Abstraction Philosophy

`sumi_platform.h` is the only module that touches the OS and windowing system.
All platform-specific code is isolated there.

The platform layer:
- Creates and manages the window
- Initializes the GPU context (OpenGL 3.3 Core by default)
- Pumps OS events and fills the input state buffers consumed by `sumi_input.h`
- Exposes a high-resolution timer
- Exposes clipboard read/write
- Exposes basic system info (screen size, DPI, locale)

The platform layer does NOT:
- Manage game-loop timing (that is `sumi_time.h`)
- Decode images or audio
- Make draw calls
- Manage assets

### Backend Isolation

The rendering backend (OpenGL, Metal, Vulkan) is selected at compile time:
```c
#define SUMI_BACKEND_OPENGL   // Default
#define SUMI_BACKEND_METAL    // macOS/iOS
#define SUMI_BACKEND_VULKAN   // Advanced users
```

Shader source strings in `sumi_shader.h` are selected per-backend. The rest
of the API surface (`sumi_gfx.h`, `sumi_texture.h`) is identical regardless
of backend.

---

## Rendering Philosophy

### 2D-First

The renderer is designed primarily for 2D. The default coordinate space is
screen-space with Y-down (origin top-left), matching image and UI conventions.
World-space (Y-up or Y-down) is controlled by the `SumiCamera2D` projection.

### Immediate-Style, Batched Internally

The public API is immediate-style: you call draw functions each frame and the
system handles batching, sorting, and flushing transparently.

```c
sumi_gfx_draw_sprite(tex, pos, src, dst, color);
sumi_gfx_draw_rect_fill(rect, color);
sumi_gfx_draw_line(a, b, thickness, color);
sumi_gfx_flush();
```

The batch breaks when:
- The texture changes
- The blend mode changes
- The shader changes
- The render target changes
- `sumi_gfx_flush()` is called explicitly

### No Scene Graph

There is no scene graph, no render queue, no node tree. You draw things in the
order you call draw functions. Layer control is your responsibility.

### Render Targets

Render targets (offscreen framebuffers) are first-class. You can bind a render
target, draw to it, then use its texture in subsequent draw calls.

### No Z-Buffer by Default

2D rendering does not use depth testing. Draw order is painter's algorithm.
Depth testing can be enabled per draw call for 2.5D effects.

---

## Error Handling Philosophy

### Philosophy

- No exceptions.
- No setjmp/longjmp.
- Errors are returned as values or via a global error state that can be queried.

### Init and resource creation functions return 0/NULL on failure

```c
SumiTextureID tex = sumi_texture_create(&desc);
if (tex == SUMI_INVALID_ID) {
    // query error
    const char* err = sumi_get_last_error();
}
```

### Global error string

```c
const char* sumi_get_last_error(void);
void        sumi_clear_error(void);
```

The error string is a static buffer. It is overwritten by the next error. It
is only meaningful immediately after a failed call.

### Debug vs Release assertions

```c
SUMI_ASSERT(cond)         // Enabled in debug; no-op in release
SUMI_ASSERT_MSG(cond, msg)
```

Fatal programmer errors (null context, double-init, invalid handle) are
asserted in debug builds. In release builds, behavior after such errors is
undefined and not handled.

### No error codes for hot-path functions

Draw functions, math functions, and query functions do not return error codes.
They are assumed to have valid input. Validation happens at creation time.

---

## Build System Expectations

Sumi has no mandatory build system. The intent is that you copy the headers
you need into your project and compile them.

### Minimal build

```sh
cc -o game main.c -I./sumi -lm
# On Windows: link opengl32.lib, gdi32.lib, winmm.lib
# On macOS: link -framework OpenGL -framework Cocoa -framework AudioToolbox
# On Linux: link -lGL -lX11 -lasound
```

### Implementation compilation

Define implementations in exactly one `.c` file:
```c
// game.c or sumi_impl.c
#define SUMI_IMPLEMENTATION   // implements everything (when using sumi.h)
// or per-module:
#define SUMI_PLATFORM_IMPLEMENTATION
#define SUMI_GFX_IMPLEMENTATION
#include "sumi.h"
```

### Compile-time configuration

All configuration is done via `#define` before the include:
```c
#define SUMI_MAX_TEXTURES       2048
#define SUMI_MAX_BATCH_QUADS    16384
#define SUMI_MAX_AUDIO_VOICES   64
#define SUMI_ARENA_DEFAULT_SIZE (4 * 1024 * 1024)
#define SUMI_LOG_LEVEL          SUMI_LOG_WARN
#define SUMI_BACKEND_OPENGL
```

### Test build

```sh
cc -DSUMI_TEST -o test_math tests/test_math.c -I./sumi -lm
```

Each module has a `#ifdef SUMI_TEST` block at the bottom of the file that
compiles a self-contained test suite.

---

## Extension & Plugin Philosophy

Sumi is extended by writing code on top of it, not by registering hooks inside it.

**There is no plugin API.** There is no callback system to inject behavior into
the framework internals.

**There is an extension pattern:** You write a new header that uses Sumi types
and functions. It does not need Sumi's permission to exist. For example:

```
sumi_tilemap.h         — ships with Sumi
sumi_ldtk.h            — user-written, parses LDtk tilemaps using sumi_tilemap
sumi_particle.h        — user-written, uses sumi_gfx draw calls
sumi_ui.h              — user-written, uses sumi_gfx, sumi_font, sumi_input
```

**The one exception is rendering backends.** If you add a new rendering
backend (Vulkan, WebGPU), you implement the internal backend interface defined
in `sumi_gfx.h`. That interface is the only formal plugin contract.

**Audio backends** follow the same pattern: implement the backend interface
struct from `sumi_audio.h`.

---

## Complete Module Breakdown

---

### `sumi.h` — Amalgamation

**Responsibility:**
Includes all modules in dependency order. Provides the `SUMI_IMPLEMENTATION`
define that triggers all module implementations at once.

**Does NOT own:**
Anything. It is a pass-through.

**Public API:**
```c
#define SUMI_IMPLEMENTATION
#include "sumi.h"
```

**Internal structure:**
```c
// sumi.h
#include "sumi_platform.h"
#include "sumi_mem.h"
#include "sumi_math.h"
#include "sumi_color.h"
#include "sumi_buf.h"
#include "sumi_map.h"
#include "sumi_event.h"
#include "sumi_time.h"
#include "sumi_input.h"
#include "sumi_image.h"
#include "sumi_asset.h"
#include "sumi_shader.h"
#include "sumi_texture.h"
#include "sumi_gfx.h"
#include "sumi_font.h"
#include "sumi_camera.h"
#include "sumi_audio.h"
#include "sumi_collision.h"
#include "sumi_debug.h"
```

**Constraints:**
- Order matters: platform → mem/math → data structures → graphics → audio → game modules
- `SUMI_IMPLEMENTATION` expands to all per-module implementation defines

---

### `sumi_platform.h` — Platform Layer

**Responsibility:**
Window creation and destruction. OpenGL/Metal context setup. Event loop pump.
Clipboard. High-resolution system timer. Screen/monitor info.

**Does NOT own:**
Input state (it fills raw event buffers that `sumi_input` consumes). Game loop
timing. Asset loading. Audio.

**Interaction with other modules:**
- Fills `_sumi_input_raw_events[]` buffer — consumed by `sumi_input_update()`
- Provides `sumi_platform_time_now()` — consumed by `sumi_time_update()`

**Key structs:**

```c
typedef struct SumiWindowDesc {
    const char* title;
    i32         width;
    i32         height;
    b8          resizable;
    b8          fullscreen;
    b8          vsync;
    b8          high_dpi;
} SumiWindowDesc;

typedef struct SumiWindowState {
    i32  width, height;         // framebuffer size
    i32  display_width;         // display size (may differ at high DPI)
    i32  display_height;
    f32  dpi_scale;
    b8   focused;
    b8   minimized;
    b8   close_requested;
} SumiWindowState;
```

**Public API:**
```c
b8               sumi_platform_init(const SumiWindowDesc* desc);
void             sumi_platform_shutdown(void);
b8               sumi_platform_poll_events(void);  // returns false when quit
void             sumi_platform_swap_buffers(void);
SumiWindowState  sumi_platform_get_window_state(void);
void             sumi_platform_set_title(const char* title);
void             sumi_platform_set_fullscreen(b8 fullscreen);
u64              sumi_platform_time_now(void);           // nanoseconds
f64              sumi_platform_time_seconds(void);
void             sumi_platform_clipboard_set(const char* text);
const char*      sumi_platform_clipboard_get(void);     // valid until next call
void             sumi_platform_show_cursor(b8 show);
void             sumi_platform_lock_cursor(b8 lock);
```

**Implementation notes:**
- Windows: Win32 (no SDL, no GLFW). OpenGL via WGL.
- macOS: Cocoa + NSOpenGLContext. Or Metal via CAMetalLayer.
- Linux: X11 + GLX. Wayland as optional compile-time alternative.
- Platform file is the only file with `#ifdef _WIN32 / __APPLE__ / __linux__` blocks.
- All platform ifdefs MUST be contained within `sumi_platform.h`.

---

### `sumi_time.h` — Timing

**Responsibility:**
Delta time tracking. Fixed timestep accumulator. Frame counter. Frame rate cap.

**Does NOT own:**
The OS timer (calls `sumi_platform_time_now()`). Event scheduling. Tweening.

**Key structs:**

```c
typedef struct SumiTimeState {
    f64  delta;          // seconds since last frame (clamped)
    f64  elapsed;        // total seconds since init
    f64  fixed_delta;    // fixed timestep duration in seconds
    f64  accumulator;    // leftover time for fixed updates
    u64  frame;          // frame counter
    f64  fps;            // smoothed FPS estimate
} SumiTimeState;
```

**Public API:**
```c
void             sumi_time_init(f64 fixed_step_hz);
void             sumi_time_update(void);             // call once per frame
const SumiTimeState* sumi_time_get(void);
b8               sumi_time_fixed_step(void);         // consume one fixed step
f64              sumi_time_delta(void);
f64              sumi_time_elapsed(void);
u64              sumi_time_frame(void);
void             sumi_time_set_max_delta(f64 max_dt); // default: 0.1s
void             sumi_time_set_fps_cap(f64 fps);       // 0 = unlimited
```

**Usage pattern:**
```c
sumi_time_init(60.0);  // 60Hz fixed step

// main loop:
sumi_time_update();
while (sumi_time_fixed_step()) {
    game_fixed_update(sumi_time_get()->fixed_delta);
}
game_update(sumi_time_delta());
game_render();
```

**Implementation notes:**
- Delta is clamped to `max_delta` (default 0.1s) to avoid spiral of death
- FPS is an exponential moving average, not a per-frame reading
- `sumi_time_fixed_step()` is a stateful loop: call until it returns false

---

### `sumi_input.h` — Input

**Responsibility:**
Keyboard state. Mouse position, buttons, scroll. Gamepad state. Touch state.
Frame-edge detection (pressed/released).

**Does NOT own:**
Raw OS events (platform fills those). Input mapping/rebinding. Text input
(IME events are exposed as a separate UTF-32 stream).

**Key structs and enums:**
```c
typedef enum SumiKey {
    SUMI_KEY_NONE = 0,
    SUMI_KEY_A, SUMI_KEY_B, /* ... */ SUMI_KEY_Z,
    SUMI_KEY_0, /* ... */ SUMI_KEY_9,
    SUMI_KEY_SPACE, SUMI_KEY_ENTER, SUMI_KEY_ESCAPE, SUMI_KEY_TAB,
    SUMI_KEY_LEFT, SUMI_KEY_RIGHT, SUMI_KEY_UP, SUMI_KEY_DOWN,
    SUMI_KEY_LSHIFT, SUMI_KEY_RSHIFT, SUMI_KEY_LCTRL, SUMI_KEY_RCTRL,
    SUMI_KEY_F1, /* ... */ SUMI_KEY_F12,
    SUMI_KEY_COUNT
} SumiKey;

typedef enum SumiMouseButton {
    SUMI_MOUSE_LEFT = 0, SUMI_MOUSE_RIGHT, SUMI_MOUSE_MIDDLE,
    SUMI_MOUSE_BUTTON_COUNT
} SumiMouseButton;

typedef struct SumiGamepad {
    f32  left_stick_x, left_stick_y;
    f32  right_stick_x, right_stick_y;
    f32  left_trigger, right_trigger;
    u32  buttons;       // bitmask of SumiGamepadButton
    b8   connected;
} SumiGamepad;

#define SUMI_MAX_GAMEPADS 4
#define SUMI_MAX_TOUCH    10
```

**Public API:**
```c
void        sumi_input_update(void);                    // call once per frame, after poll_events

// Keyboard
b8          sumi_input_key_down(SumiKey key);
b8          sumi_input_key_pressed(SumiKey key);        // true for exactly one frame
b8          sumi_input_key_released(SumiKey key);
b8          sumi_input_key_mod(SumiKeyMod mod);         // shift/ctrl/alt

// Mouse
SumiVec2    sumi_input_mouse_pos(void);
SumiVec2    sumi_input_mouse_delta(void);
f32         sumi_input_mouse_scroll(void);
b8          sumi_input_mouse_down(SumiMouseButton btn);
b8          sumi_input_mouse_pressed(SumiMouseButton btn);
b8          sumi_input_mouse_released(SumiMouseButton btn);

// Text input
b8          sumi_input_text_next(u32* codepoint);       // iterate UTF-32 chars this frame

// Gamepad
const SumiGamepad* sumi_input_gamepad(i32 index);
b8          sumi_input_gamepad_button_down(i32 index, SumiGamepadButton btn);
b8          sumi_input_gamepad_button_pressed(i32 index, SumiGamepadButton btn);
```

**Implementation notes:**
- Two frames of key state (current, previous) stored as bitsets `u8[SUMI_KEY_COUNT / 8]`
- `pressed` = current & ~previous; `released` = ~current & previous
- Mouse position is in window pixel coordinates (not world space)
- Scroll is accumulated over the frame, reset in `sumi_input_update()`
- Text input is a small fixed ring buffer of UTF-32 codepoints

---

### `sumi_mem.h` — Memory Allocators

**Responsibility:**
Arena allocator. Pool allocator. Scratch allocator (per-frame temp memory).
The default system allocator wrapper.

**Does NOT own:**
Memory for specific subsystems (each subsystem has its own arena or takes an
allocator pointer).

**Key structs:**
```c
typedef struct SumiArena {
    u8*   base;
    usize size;
    usize offset;
    usize prev_offset;  // for single undo
} SumiArena;

typedef struct SumiPool {
    u8*   base;
    usize element_size;
    usize capacity;
    void* free_list;    // internal linked free list
    u32   count;
} SumiPool;
```

**Public API:**
```c
// Arena
SumiArena   sumi_arena_create(usize size, SumiAllocator* alloc);
void        sumi_arena_destroy(SumiArena* a);
void*       sumi_arena_push(SumiArena* a, usize size);
void*       sumi_arena_push_zero(SumiArena* a, usize size);
void        sumi_arena_pop(SumiArena* a, usize size);
usize       sumi_arena_save(SumiArena* a);
void        sumi_arena_restore(SumiArena* a, usize checkpoint);
void        sumi_arena_reset(SumiArena* a);

// Pool
SumiPool    sumi_pool_create(usize element_size, usize capacity, SumiAllocator* alloc);
void        sumi_pool_destroy(SumiPool* p);
void*       sumi_pool_alloc(SumiPool* p);
void        sumi_pool_free(SumiPool* p, void* ptr);
void        sumi_pool_reset(SumiPool* p);

// Scratch (two alternating arenas — safe to use across function calls in same frame)
void*       sumi_scratch_push(usize size);
void        sumi_scratch_reset(void);            // call at end of frame
void        sumi_scratch_init(usize size_each);  // call at startup

// System allocator
void*       sumi_malloc(usize size);
void*       sumi_realloc(void* ptr, usize size);
void        sumi_free(void* ptr);

// Convenience macro
#define SUMI_PUSH_STRUCT(arena, T)       ((T*)sumi_arena_push_zero(arena, sizeof(T)))
#define SUMI_PUSH_ARRAY(arena, T, count) ((T*)sumi_arena_push_zero(arena, sizeof(T)*(count)))
```

**Implementation notes:**
- Arena push aligns to 8 bytes by default; use `sumi_arena_push_aligned` for SIMD
- Pool free list is embedded in the freed elements themselves (no extra metadata)
- Scratch uses two arenas alternated per-frame to avoid invalidating last frame's temp pointers
- `sumi_scratch_*` functions access a module-global state initialized once

---

### `sumi_math.h` — Math

**Responsibility:**
All math types and operations needed for 2D (and basic 3D) game development.

**Does NOT own:**
Random number generation. Complex number physics simulation. Pathfinding math.

**Types:**
```c
typedef struct SumiVec2   { f32 x, y; }          SumiVec2;
typedef struct SumiVec3   { f32 x, y, z; }        SumiVec3;
typedef struct SumiVec4   { f32 x, y, z, w; }     SumiVec4;
typedef struct SumiIVec2  { i32 x, y; }           SumiIVec2;
typedef struct SumiIVec3  { i32 x, y, z; }        SumiIVec3;
typedef struct SumiRect   { f32 x, y, w, h; }     SumiRect;
typedef struct SumiIRect  { i32 x, y, w, h; }     SumiIRect;
typedef struct SumiMat3   { f32 m[3][3]; }         SumiMat3;
typedef struct SumiMat4   { f32 m[4][4]; }         SumiMat4;

typedef struct SumiTransform2D {
    SumiVec2 position;
    SumiVec2 scale;
    f32      rotation;   // radians
} SumiTransform2D;
```

**Public API (selected):**
```c
// Vec2
SumiVec2  sumi_v2(f32 x, f32 y);
SumiVec2  sumi_v2_add(SumiVec2 a, SumiVec2 b);
SumiVec2  sumi_v2_sub(SumiVec2 a, SumiVec2 b);
SumiVec2  sumi_v2_mul(SumiVec2 v, f32 s);
SumiVec2  sumi_v2_div(SumiVec2 v, f32 s);
f32       sumi_v2_dot(SumiVec2 a, SumiVec2 b);
f32       sumi_v2_cross(SumiVec2 a, SumiVec2 b);
f32       sumi_v2_len(SumiVec2 v);
f32       sumi_v2_len2(SumiVec2 v);
SumiVec2  sumi_v2_norm(SumiVec2 v);
SumiVec2  sumi_v2_lerp(SumiVec2 a, SumiVec2 b, f32 t);
SumiVec2  sumi_v2_rotate(SumiVec2 v, f32 angle);
SumiVec2  sumi_v2_perp(SumiVec2 v);           // (-y, x)
f32       sumi_v2_angle(SumiVec2 v);
f32       sumi_v2_dist(SumiVec2 a, SumiVec2 b);
b8        sumi_v2_eq(SumiVec2 a, SumiVec2 b);

// Rect
SumiRect  sumi_rect(f32 x, f32 y, f32 w, f32 h);
SumiRect  sumi_rect_from_center(SumiVec2 center, SumiVec2 size);
SumiVec2  sumi_rect_center(SumiRect r);
SumiVec2  sumi_rect_size(SumiRect r);
b8        sumi_rect_contains_point(SumiRect r, SumiVec2 p);
b8        sumi_rect_overlaps(SumiRect a, SumiRect b);
SumiRect  sumi_rect_intersect(SumiRect a, SumiRect b);
SumiRect  sumi_rect_union(SumiRect a, SumiRect b);
SumiRect  sumi_rect_expand(SumiRect r, f32 amount);
SumiRect  sumi_rect_offset(SumiRect r, SumiVec2 offset);

// Mat3 (2D transforms)
SumiMat3  sumi_mat3_identity(void);
SumiMat3  sumi_mat3_translate(SumiVec2 t);
SumiMat3  sumi_mat3_rotate(f32 angle);
SumiMat3  sumi_mat3_scale(SumiVec2 s);
SumiMat3  sumi_mat3_mul(SumiMat3 a, SumiMat3 b);
SumiVec2  sumi_mat3_transform_point(SumiMat3 m, SumiVec2 p);
SumiVec2  sumi_mat3_transform_vec(SumiMat3 m, SumiVec2 v);
SumiMat3  sumi_mat3_inverse(SumiMat3 m);
SumiMat4  sumi_mat3_to_mat4(SumiMat3 m);

// Transform2D
SumiMat3  sumi_transform2d_to_mat3(SumiTransform2D t);
SumiTransform2D sumi_transform2d_combine(SumiTransform2D parent, SumiTransform2D child);

// Scalar utils
f32  sumi_lerp(f32 a, f32 b, f32 t);
f32  sumi_clamp(f32 v, f32 lo, f32 hi);
f32  sumi_clamp01(f32 v);
f32  sumi_smoothstep(f32 edge0, f32 edge1, f32 x);
f32  sumi_remap(f32 v, f32 in_lo, f32 in_hi, f32 out_lo, f32 out_hi);
f32  sumi_wrap(f32 v, f32 lo, f32 hi);
f32  sumi_sign(f32 v);
f32  sumi_snap(f32 v, f32 grid);
b8   sumi_nearly_eq(f32 a, f32 b, f32 eps);
```

**Implementation notes:**
- All functions are static inline — no .c compilation needed for math
- No SIMD intrinsics in the public API; a `sumi_math_simd.h` extension may provide them
- Mat3 is used for all 2D transforms; Mat4 is provided for 3D interop only

---

### `sumi_color.h` — Color

**Responsibility:**
Color types, constructors, conversion, blending, and predefined palette.

**Does NOT own:**
Rendering. Shaders. Image processing beyond pixel-level math.

**Types:**
```c
typedef struct SumiColor  { u8  r, g, b, a; }         SumiColor;   // 32-bit packed
typedef struct SumiColorF { f32 r, g, b, a; }         SumiColorF;  // float
typedef u32               SumiColorU32;                              // 0xAARRGGBB
```

**Public API:**
```c
SumiColor   sumi_color(u8 r, u8 g, u8 b, u8 a);
SumiColor   sumi_color_hex(u32 hex);              // 0xRRGGBBAA
SumiColor   sumi_color_hsv(f32 h, f32 s, f32 v); // h: 0-360, s/v: 0-1
SumiColor   sumi_color_lerp(SumiColor a, SumiColor b, f32 t);
SumiColor   sumi_color_fade(SumiColor c, f32 alpha);
SumiColor   sumi_color_tint(SumiColor c, SumiColor tint);
SumiColorF  sumi_color_to_float(SumiColor c);
SumiColor   sumi_color_from_float(SumiColorF c);
SumiColorU32 sumi_color_to_u32(SumiColor c);

// Predefined
#define SUMI_WHITE    (sumi_color(255,255,255,255))
#define SUMI_BLACK    (sumi_color(0,0,0,255))
#define SUMI_RED      (sumi_color(255,0,0,255))
#define SUMI_GREEN    (sumi_color(0,255,0,255))
#define SUMI_BLUE     (sumi_color(0,0,255,255))
#define SUMI_YELLOW   (sumi_color(255,255,0,255))
#define SUMI_CYAN     (sumi_color(0,255,255,255))
#define SUMI_MAGENTA  (sumi_color(255,0,255,255))
#define SUMI_BLANK    (sumi_color(0,0,0,0))
```

---

### `sumi_buf.h` — Buffers

**Responsibility:**
Type-safe dynamic arrays (stretchy buffers) via macros. Fixed ring buffer.
String builder.

**Does NOT own:**
Hash maps. Trees. Sorted structures.

**Public API:**
```c
// Dynamic array — T* used directly as the array, metadata stored before the pointer
#define sumi_buf_push(buf, item)        // append item, grow if needed
#define sumi_buf_pop(buf)               // remove last item
#define sumi_buf_count(buf)             // number of elements
#define sumi_buf_capacity(buf)          // allocated capacity
#define sumi_buf_reserve(buf, n)        // ensure space for n more
#define sumi_buf_clear(buf)             // set count to 0 (no free)
#define sumi_buf_free(buf)              // release memory
#define sumi_buf_last(buf)              // pointer to last element
#define sumi_buf_insert(buf, i, item)   // insert at index i
#define sumi_buf_remove_swap(buf, i)    // remove by swap with last (O(1))

// Ring buffer (fixed capacity, typed)
typedef struct SumiRingBuf {
    u8*   data;
    usize element_size;
    u32   capacity;
    u32   head, tail, count;
} SumiRingBuf;

SumiRingBuf sumi_ringbuf_create(usize element_size, u32 capacity, SumiAllocator* a);
void        sumi_ringbuf_destroy(SumiRingBuf* rb);
b8          sumi_ringbuf_push(SumiRingBuf* rb, const void* item);
b8          sumi_ringbuf_pop(SumiRingBuf* rb, void* out);
b8          sumi_ringbuf_peek(const SumiRingBuf* rb, void* out);
b8          sumi_ringbuf_is_full(const SumiRingBuf* rb);
b8          sumi_ringbuf_is_empty(const SumiRingBuf* rb);
void        sumi_ringbuf_clear(SumiRingBuf* rb);

// String builder
typedef struct SumiStrBuf {
    char* data;
    usize len;
    usize capacity;
    SumiAllocator* alloc;
} SumiStrBuf;

SumiStrBuf  sumi_strbuf_create(usize initial_cap, SumiAllocator* a);
void        sumi_strbuf_destroy(SumiStrBuf* sb);
void        sumi_strbuf_append(SumiStrBuf* sb, const char* str);
void        sumi_strbuf_appendf(SumiStrBuf* sb, const char* fmt, ...);
void        sumi_strbuf_append_char(SumiStrBuf* sb, char c);
void        sumi_strbuf_clear(SumiStrBuf* sb);
const char* sumi_strbuf_cstr(const SumiStrBuf* sb);
```

**Implementation notes:**
- Dynamic array stores count and capacity as `u32` fields immediately before the data pointer
- Growth factor is 1.5x to reduce over-allocation
- `sumi_buf_*` macros work on any pointer type (`Foo* buf = NULL; sumi_buf_push(buf, item)`)
- The null pointer is a valid empty buffer

---

### `sumi_map.h` — Hash Map & Set

**Responsibility:**
Open-addressing hash map (string→void* and u64→u64 variants). Hash set.

**Does NOT own:**
Sorted maps. Graphs. Specialized data structures.

**Types:**
```c
// String key → void* value (common case for named assets, etc.)
typedef struct SumiStrMap { ... } SumiStrMap;

// u64 key → u64 value (handles, IDs, etc.)
typedef struct SumiU64Map { ... } SumiU64Map;

// u64 hash set
typedef struct SumiU64Set { ... } SumiU64Set;
```

**Public API:**
```c
// SumiStrMap
SumiStrMap  sumi_strmap_create(u32 initial_cap, SumiAllocator* a);
void        sumi_strmap_destroy(SumiStrMap* m);
void        sumi_strmap_set(SumiStrMap* m, const char* key, void* value);
void*       sumi_strmap_get(const SumiStrMap* m, const char* key);    // NULL if missing
b8          sumi_strmap_has(const SumiStrMap* m, const char* key);
b8          sumi_strmap_remove(SumiStrMap* m, const char* key);
void        sumi_strmap_clear(SumiStrMap* m);
u32         sumi_strmap_count(const SumiStrMap* m);

// SumiU64Map
SumiU64Map  sumi_u64map_create(u32 initial_cap, SumiAllocator* a);
void        sumi_u64map_destroy(SumiU64Map* m);
void        sumi_u64map_set(SumiU64Map* m, u64 key, u64 value);
b8          sumi_u64map_get(const SumiU64Map* m, u64 key, u64* out);
b8          sumi_u64map_has(const SumiU64Map* m, u64 key);
b8          sumi_u64map_remove(SumiU64Map* m, u64 key);
void        sumi_u64map_clear(SumiU64Map* m);

// Iteration (order not guaranteed)
typedef void (*SumiStrMapIterFn)(const char* key, void* value, void* ctx);
void        sumi_strmap_each(const SumiStrMap* m, SumiStrMapIterFn fn, void* ctx);
```

**Implementation notes:**
- Open addressing with Robin Hood probing
- Load factor 0.75; doubles capacity at threshold
- String keys are interned (copied into internal arena on insert)
- Tombstone approach for deletions; rehash triggered if tombstones exceed 25%

---

### `sumi_event.h` — Event Queue

**Responsibility:**
A typed, synchronous event queue. Register event types, push events, dispatch
all pending events to registered listeners.

**Does NOT own:**
Async messaging. Thread-safe queuing (extend separately if needed). Complex
routing or filtering.

**Design:**
Events are u32-tagged payloads. The payload is a fixed-size `u8[64]` block
(compile-time configurable). Dispatch is synchronous — all listeners are
called before `sumi_event_dispatch` returns.

**Types:**
```c
#define SUMI_EVENT_PAYLOAD_SIZE 64

typedef u32 SumiEventType;

typedef struct SumiEvent {
    SumiEventType type;
    u8            data[SUMI_EVENT_PAYLOAD_SIZE];
} SumiEvent;

typedef void (*SumiEventListenerFn)(const SumiEvent* e, void* ctx);

typedef struct SumiEventHandle { u32 id; } SumiEventHandle;
```

**Public API:**
```c
void             sumi_event_init(u32 queue_capacity, SumiAllocator* a);
void             sumi_event_shutdown(void);

SumiEventType    sumi_event_register(const char* name);    // returns unique type id
SumiEventType    sumi_event_find(const char* name);        // 0 if not found

void             sumi_event_push(SumiEventType type, const void* data, usize data_size);
void             sumi_event_dispatch(void);                // call once per frame

SumiEventHandle  sumi_event_listen(SumiEventType type, SumiEventListenerFn fn, void* ctx);
void             sumi_event_unlisten(SumiEventHandle handle);
```

**Usage pattern:**
```c
// At startup
SumiEventType EV_PLAYER_DIED = sumi_event_register("player_died");

// Somewhere in game logic
typedef struct { i32 player_id; } PlayerDiedData;
PlayerDiedData payload = { .player_id = 1 };
sumi_event_push(EV_PLAYER_DIED, &payload, sizeof(payload));

// Listener (registered at init)
void on_player_died(const SumiEvent* e, void* ctx) {
    PlayerDiedData* d = (PlayerDiedData*)e->data;
    // handle event
}

// Per frame
sumi_event_dispatch();
```

**Implementation notes:**
- Queue is a ring buffer; overflow is a programmer error (assert in debug)
- Listeners per event type are stored in a small dynamic array
- Events pushed during dispatch are deferred to the next `sumi_event_dispatch` call

---

### `sumi_image.h` — Image Decode/Encode

**Responsibility:**
CPU-side image loading and saving. Decode PNG, BMP, and QOI from memory.
Encode PNG and QOI to memory. Basic pixel operations.

**Does NOT own:**
GPU texture upload (that is `sumi_texture.h`). File I/O (that is `sumi_asset.h`).
Hardware-accelerated image processing.

**Types:**
```c
typedef enum SumiPixelFormat {
    SUMI_PIXEL_R8    = 1,
    SUMI_PIXEL_RG8   = 2,
    SUMI_PIXEL_RGB8  = 3,
    SUMI_PIXEL_RGBA8 = 4,
} SumiPixelFormat;

typedef struct SumiImage {
    u8*            pixels;      // heap-allocated; caller must free with sumi_image_free
    i32            width;
    i32            height;
    SumiPixelFormat format;
} SumiImage;
```

**Public API:**
```c
// Decode
SumiImage   sumi_image_decode(const u8* data, usize size);        // auto-detect format
SumiImage   sumi_image_decode_png(const u8* data, usize size);
SumiImage   sumi_image_decode_bmp(const u8* data, usize size);
SumiImage   sumi_image_decode_qoi(const u8* data, usize size);

// Encode (returns heap-allocated buffer; caller frees with sumi_free)
u8*         sumi_image_encode_png(const SumiImage* img, usize* out_size);
u8*         sumi_image_encode_qoi(const SumiImage* img, usize* out_size);

// Create / copy
SumiImage   sumi_image_create(i32 w, i32 h, SumiPixelFormat fmt);
SumiImage   sumi_image_clone(const SumiImage* src);
void        sumi_image_free(SumiImage* img);

// Pixel access
SumiColor   sumi_image_get_pixel(const SumiImage* img, i32 x, i32 y);
void        sumi_image_set_pixel(SumiImage* img, i32 x, i32 y, SumiColor c);

// Operations (in-place)
void        sumi_image_flip_v(SumiImage* img);
void        sumi_image_flip_h(SumiImage* img);
void        sumi_image_convert(SumiImage* img, SumiPixelFormat target_fmt);
SumiImage   sumi_image_crop(const SumiImage* img, SumiIRect region);
```

**Implementation notes:**
- PNG decoder is a minimal implementation: deflate/inflate + PNG filter passes
- QOI is the preferred fast format (simple to implement, fast to decode)
- BMP decoder covers uncompressed 24/32-bit only
- No JPEG support by default (lossy, complex); JPEG can be added via extension

---

### `sumi_asset.h` — Asset I/O

**Responsibility:**
Synchronous file reading. Memory-mapped file access. Virtual filesystem mount
points. Simple asset caching by path.

**Does NOT own:**
Image/audio decoding. GPU upload. Async loading. Compression. Hot reload.

**Types:**
```c
typedef struct SumiFileData {
    u8*   data;     // heap-allocated; free with sumi_asset_free
    usize size;
} SumiFileData;

typedef struct SumiVFS { ... } SumiVFS;   // opaque
```

**Public API:**
```c
// Raw file I/O (no VFS)
SumiFileData sumi_asset_read_file(const char* path);
b8           sumi_asset_write_file(const char* path, const u8* data, usize size);
void         sumi_asset_free(SumiFileData* fd);
b8           sumi_asset_file_exists(const char* path);
usize        sumi_asset_file_size(const char* path);

// Virtual filesystem (optional, must init)
void         sumi_vfs_init(void);
void         sumi_vfs_shutdown(void);
void         sumi_vfs_mount(const char* virtual_path, const char* real_path);
void         sumi_vfs_unmount(const char* virtual_path);
SumiFileData sumi_vfs_read(const char* virtual_path);

// Asset cache (optional layer on top of VFS)
void         sumi_asset_cache_init(SumiAllocator* a);
void         sumi_asset_cache_shutdown(void);
const u8*    sumi_asset_cache_get(const char* path, usize* out_size); // NULL if not cached
void         sumi_asset_cache_store(const char* path, const u8* data, usize size);
void         sumi_asset_cache_evict(const char* path);
void         sumi_asset_cache_clear(void);
```

**Usage pattern:**
```c
sumi_vfs_mount("/assets", "./res");   // map virtual /assets to ./res on disk

SumiFileData fd = sumi_vfs_read("/assets/sprites/player.png");
SumiImage img = sumi_image_decode(fd.data, fd.size);
sumi_asset_free(&fd);
// Now upload img to GPU via sumi_texture_create(...)
```

**Implementation notes:**
- VFS mount table is a flat array of (virtual_prefix, real_prefix) pairs
- Overlapping mounts are checked in reverse-registration order
- Memory-mapped files are a compile-time opt-in (`SUMI_ASSET_MMAP`)
- The asset cache stores reference-counted blobs; it does not interpret the data

---

### `sumi_texture.h` — GPU Textures

**Responsibility:**
GPU texture creation, destruction, and sampler configuration. Texture atlas
management. Sub-texture (sprite region) definitions.

**Does NOT own:**
Image file decoding (`sumi_image.h`). File I/O (`sumi_asset.h`). Font atlases
(`sumi_font.h` owns those).

**Types:**
```c
typedef u32 SumiTextureID;
#define SUMI_INVALID_TEXTURE 0

typedef enum SumiPixelFilter {
    SUMI_FILTER_NEAREST,
    SUMI_FILTER_LINEAR,
} SumiPixelFilter;

typedef enum SumiTextureWrap {
    SUMI_WRAP_CLAMP,
    SUMI_WRAP_REPEAT,
    SUMI_WRAP_MIRROR,
} SumiTextureWrap;

typedef struct SumiTextureDesc {
    const u8*       data;       // pixel data (copied to GPU; may be NULL for blank)
    i32             width;
    i32             height;
    SumiPixelFormat format;
    SumiPixelFilter filter;
    SumiTextureWrap wrap;
    b8              generate_mipmaps;
} SumiTextureDesc;

typedef struct SumiTextureInfo {
    i32             width, height;
    SumiPixelFormat format;
    SumiPixelFilter filter;
} SumiTextureInfo;

typedef struct SumiSubTexture {
    SumiTextureID   texture;
    SumiRect        uv;         // normalized UV coordinates (0..1)
    SumiIRect       pixel_rect; // source pixel rect in the atlas
} SumiSubTexture;

// Atlas
typedef struct SumiAtlas { ... } SumiAtlas;  // opaque
```

**Public API:**
```c
// Texture
SumiTextureID    sumi_texture_create(const SumiTextureDesc* desc);
void             sumi_texture_destroy(SumiTextureID id);
void             sumi_texture_update(SumiTextureID id, const u8* data, i32 x, i32 y, i32 w, i32 h);
SumiTextureInfo  sumi_texture_get_info(SumiTextureID id);
b8               sumi_texture_valid(SumiTextureID id);

// Convenience
SumiTextureID    sumi_texture_from_image(const SumiImage* img, SumiPixelFilter filter);
SumiTextureID    sumi_texture_create_blank(i32 w, i32 h, SumiColor color);

// Sub-texture
SumiSubTexture   sumi_subtexture(SumiTextureID id, SumiIRect pixel_rect);

// Texture atlas (runtime packing)
SumiAtlas*       sumi_atlas_create(i32 width, i32 height, SumiPixelFormat fmt, SumiAllocator* a);
void             sumi_atlas_destroy(SumiAtlas* atlas);
SumiSubTexture   sumi_atlas_pack(SumiAtlas* atlas, const SumiImage* img, const char* name);
SumiSubTexture   sumi_atlas_find(const SumiAtlas* atlas, const char* name);
SumiTextureID    sumi_atlas_get_texture(const SumiAtlas* atlas); // uploads if dirty
void             sumi_atlas_reset(SumiAtlas* atlas);
```

**Implementation notes:**
- Internal texture table: fixed-size array indexed by handle ID
- GPU handle (GLuint or equivalent) stored alongside width/height/format
- Atlas uses a shelf/skyline packing algorithm (simple, good enough for 2D)
- Atlas upload is lazy: `sumi_atlas_get_texture()` uploads if pixels changed since last call

---

### `sumi_shader.h` — Shaders

**Responsibility:**
Shader compilation from source strings. Uniform binding. Built-in shader library.

**Does NOT own:**
Texture binding (that is `sumi_gfx.h`). Pipeline state beyond shader. Render targets.

**Types:**
```c
typedef u32 SumiShaderID;
#define SUMI_INVALID_SHADER 0
```

**Public API:**
```c
SumiShaderID  sumi_shader_create(const char* vert_src, const char* frag_src);
SumiShaderID  sumi_shader_create_from_files(const char* vert_path, const char* frag_path);
void          sumi_shader_destroy(SumiShaderID id);
b8            sumi_shader_valid(SumiShaderID id);

// Uniforms (shader must be active: call from sumi_gfx_set_shader or within batch)
void  sumi_shader_set_int(SumiShaderID id, const char* name, i32 val);
void  sumi_shader_set_float(SumiShaderID id, const char* name, f32 val);
void  sumi_shader_set_vec2(SumiShaderID id, const char* name, SumiVec2 val);
void  sumi_shader_set_vec4(SumiShaderID id, const char* name, SumiVec4 val);
void  sumi_shader_set_mat3(SumiShaderID id, const char* name, SumiMat3 val);
void  sumi_shader_set_mat4(SumiShaderID id, const char* name, SumiMat4 val);
void  sumi_shader_set_texture_slot(SumiShaderID id, const char* name, i32 slot);

// Built-in shaders (always available)
SumiShaderID  sumi_shader_default(void);       // sprite batch shader
SumiShaderID  sumi_shader_sdf(void);           // signed distance field text
SumiShaderID  sumi_shader_screen(void);        // full-screen quad / post-process base
```

**Implementation notes:**
- Uniform locations are cached in a small hash map per shader (string → GLint)
- Built-in shaders are stored as string literals selected per backend at compile time
- Shader compilation errors write to `sumi_get_last_error()` and the debug log
- No shader hot-reload. That is an extension responsibility.

---

### `sumi_gfx.h` — Renderer

**Responsibility:**
The core 2D renderer. Draw calls (sprites, rects, lines, circles, polygons,
custom meshes). Batching. Render targets. Blend modes. Layer/z-sorting if
explicitly requested.

**Does NOT own:**
Windowing. Texture loading. Font rendering (fonts call into gfx). Camera math.
Asset loading.

**Types:**
```c
typedef u32 SumiRenderTargetID;
#define SUMI_INVALID_RENDER_TARGET 0

typedef enum SumiBlendMode {
    SUMI_BLEND_NONE,
    SUMI_BLEND_ALPHA,
    SUMI_BLEND_ADD,
    SUMI_BLEND_MULTIPLY,
    SUMI_BLEND_SCREEN,
} SumiBlendMode;

typedef struct SumiGfxConfig {
    i32   max_batch_quads;      // default: 8192
    i32   max_textures;         // default: 512
    i32   max_render_targets;   // default: 32
    b8    premultiplied_alpha;  // default: false
} SumiGfxConfig;

typedef struct SumiDrawConfig {
    SumiColor     tint;
    SumiBlendMode blend;
    SumiShaderID  shader;       // SUMI_INVALID_SHADER = use default
    i32           z;            // sort key (used if sort mode is on)
    SumiMat3      transform;    // extra transform (identity = no extra transform)
} SumiDrawConfig;
```

**Public API:**
```c
// Init/shutdown
b8    sumi_gfx_init(const SumiGfxConfig* config);
void  sumi_gfx_shutdown(void);

// Frame
void  sumi_gfx_begin_frame(void);
void  sumi_gfx_end_frame(void);       // implicitly flushes
void  sumi_gfx_flush(void);           // explicit flush
void  sumi_gfx_clear(SumiColor color);

// Viewport
void  sumi_gfx_set_viewport(i32 x, i32 y, i32 w, i32 h);
void  sumi_gfx_set_projection(SumiMat4 proj);   // set via sumi_camera2d_get_projection()

// Render target
SumiRenderTargetID  sumi_render_target_create(i32 w, i32 h, SumiPixelFormat fmt);
void                sumi_render_target_destroy(SumiRenderTargetID id);
void                sumi_gfx_set_render_target(SumiRenderTargetID id);  // 0 = screen
SumiTextureID       sumi_render_target_get_texture(SumiRenderTargetID id);
void                sumi_gfx_read_pixels(SumiRect region, SumiPixelFormat fmt, u8* out);

// Blend / shader / scissor
void  sumi_gfx_set_blend_mode(SumiBlendMode mode);
void  sumi_gfx_set_shader(SumiShaderID id);
void  sumi_gfx_set_scissor(SumiRect rect);
void  sumi_gfx_clear_scissor(void);

// Draw calls — all accumulate into current batch
void  sumi_gfx_draw_texture(SumiTextureID tex, SumiVec2 pos, SumiColor tint);
void  sumi_gfx_draw_subtexture(SumiSubTexture sub, SumiVec2 pos, SumiColor tint);
void  sumi_gfx_draw_sprite(SumiTextureID tex, SumiRect src, SumiRect dst,
                            SumiVec2 origin, f32 rotation, SumiColor tint);
void  sumi_gfx_draw_sprite_sub(SumiSubTexture sub, SumiRect dst,
                                SumiVec2 origin, f32 rotation, SumiColor tint);
void  sumi_gfx_draw_rect_fill(SumiRect rect, SumiColor color);
void  sumi_gfx_draw_rect_outline(SumiRect rect, f32 thickness, SumiColor color);
void  sumi_gfx_draw_rect_ex(SumiRect dst, SumiVec2 origin, f32 rotation, SumiColor color);
void  sumi_gfx_draw_circle_fill(SumiVec2 center, f32 radius, i32 segments, SumiColor color);
void  sumi_gfx_draw_circle_outline(SumiVec2 center, f32 radius, i32 segments, f32 thickness, SumiColor color);
void  sumi_gfx_draw_line(SumiVec2 a, SumiVec2 b, f32 thickness, SumiColor color);
void  sumi_gfx_draw_triangle(SumiVec2 a, SumiVec2 b, SumiVec2 c, SumiColor color);
void  sumi_gfx_draw_polygon(const SumiVec2* verts, i32 count, SumiColor color);
void  sumi_gfx_draw_polyline(const SumiVec2* verts, i32 count, f32 thickness,
                              b8 closed, SumiColor color);

// Advanced draw config variants
void  sumi_gfx_draw_sprite_ex(SumiTextureID tex, SumiRect src, SumiRect dst,
                               SumiVec2 origin, f32 rotation, const SumiDrawConfig* cfg);

// Stats
typedef struct SumiGfxStats {
    u32 draw_calls;
    u32 quads;
    u32 texture_binds;
    u32 shader_binds;
    u32 flush_count;
} SumiGfxStats;
SumiGfxStats sumi_gfx_get_stats(void);
```

**Implementation notes:**
- Batch vertex layout: `vec2 pos, vec2 uv, vec4 color (packed u32)`
- Batch is a VBO with pre-allocated storage; mapped with persistent mapping if available
- Texture slot array: up to 8 simultaneous textures in one batch (uses sampler2D array in shader)
- White 1×1 texture used for shapes (no texture binding change for solid-color shapes)
- Draw calls that would exceed batch capacity trigger an auto-flush

---

### `sumi_font.h` — Font & Text

**Responsibility:**
Font atlas generation from TTF/OTF data. Glyph metrics. Text measurement and
draw calls. SDF font support.

**Does NOT own:**
Full text layout engine (BiDi, shaping, ligatures). Internationalization beyond
basic Unicode codepoint rendering.

**Types:**
```c
typedef u32 SumiFontID;
#define SUMI_INVALID_FONT 0

typedef struct SumiFontDesc {
    const u8*  data;            // TTF/OTF file data (Sumi copies internally)
    usize      data_size;
    f32        size_px;         // base font size in pixels
    i32        atlas_width;     // default: 512
    i32        atlas_height;    // default: 512
    b8         sdf;             // render as SDF
    const u32* codepoints;      // NULL = ASCII 32-126
    i32        codepoint_count; // 0 = default
} SumiFontDesc;

typedef struct SumiGlyph {
    SumiSubTexture  sub;
    SumiVec2        offset;      // pen offset
    f32             advance;     // pen advance
} SumiGlyph;

typedef struct SumiTextMetrics {
    f32  width;
    f32  height;
    f32  ascent;
    f32  descent;
    f32  line_height;
    i32  line_count;
} SumiTextMetrics;
```

**Public API:**
```c
SumiFontID       sumi_font_create(const SumiFontDesc* desc);
void             sumi_font_destroy(SumiFontID id);
b8               sumi_font_valid(SumiFontID id);

const SumiGlyph* sumi_font_get_glyph(SumiFontID id, u32 codepoint);
f32              sumi_font_get_kerning(SumiFontID id, u32 left, u32 right);
SumiTextMetrics  sumi_font_measure(SumiFontID id, const char* text, f32 size_px);
SumiTextureID    sumi_font_get_atlas_texture(SumiFontID id);

// Draw
void  sumi_font_draw(SumiFontID id, const char* text, SumiVec2 pos,
                     f32 size_px, SumiColor color);
void  sumi_font_draw_wrapped(SumiFontID id, const char* text, SumiRect bounds,
                              f32 size_px, SumiColor color);
void  sumi_font_draw_aligned(SumiFontID id, const char* text, SumiVec2 pos,
                              f32 size_px, SumiColor color,
                              f32 align_x, f32 align_y); // 0=left/top, 0.5=center, 1=right/bottom
```

**Implementation notes:**
- Font rasterization uses stb_truetype internally (bundled, no external dep)
- Atlas uses a shelf packer, same as `sumi_atlas_*`
- Multiple font sizes from the same TTF data require separate `SumiFontID` instances
- SDF fonts use a different shader (`sumi_shader_sdf()`) — `sumi_font_draw` sets it automatically

---

### `sumi_camera.h` — 2D Camera

**Responsibility:**
Camera2D struct and transform logic. World↔screen coordinate conversion.
Viewport-relative projection matrix generation. Camera shake.

**Does NOT own:**
Rendering. Any GPU state. Input processing.

**Types:**
```c
typedef struct SumiCamera2D {
    SumiVec2  target;       // world position the camera looks at
    SumiVec2  offset;       // screen-space offset of the target (e.g. screen center)
    f32       zoom;
    f32       rotation;     // radians
} SumiCamera2D;

typedef struct SumicameraShake {
    f32  magnitude;
    f32  frequency;
    f32  duration;
    f32  _elapsed;
    f32  _seed;
} SumiCameraShake;
```

**Public API:**
```c
SumiCamera2D  sumi_camera2d_default(void);
SumiMat4      sumi_camera2d_get_projection(const SumiCamera2D* cam,
                                            f32 viewport_w, f32 viewport_h);
SumiVec2      sumi_camera2d_world_to_screen(const SumiCamera2D* cam,
                                             SumiVec2 world_pos,
                                             f32 viewport_w, f32 viewport_h);
SumiVec2      sumi_camera2d_screen_to_world(const SumiCamera2D* cam,
                                             SumiVec2 screen_pos,
                                             f32 viewport_w, f32 viewport_h);
SumiRect      sumi_camera2d_get_world_bounds(const SumiCamera2D* cam,
                                              f32 viewport_w, f32 viewport_h);
void          sumi_camera2d_follow(SumiCamera2D* cam, SumiVec2 target,
                                   f32 lerp_speed, f32 dt);

// Camera shake
SumiCameraShake  sumi_camera_shake_create(f32 magnitude, f32 frequency, f32 duration);
void             sumi_camera_shake_update(SumiCameraShake* shake, SumiCamera2D* cam, f32 dt);
b8               sumi_camera_shake_active(const SumiCameraShake* shake);
```

**Usage pattern:**
```c
SumiCamera2D cam = sumi_camera2d_default();
cam.offset = sumi_v2(screen_w * 0.5f, screen_h * 0.5f);
cam.zoom = 2.0f;
cam.target = player_pos;

// Each frame:
SumiMat4 proj = sumi_camera2d_get_projection(&cam, screen_w, screen_h);
sumi_gfx_set_projection(proj);
// now all draw calls are in world space

// Convert mouse position to world
SumiVec2 mouse_world = sumi_camera2d_screen_to_world(
    &cam, sumi_input_mouse_pos(), screen_w, screen_h);
```

---

### `sumi_audio.h` — Audio

**Responsibility:**
Audio device initialization. Sound loading from memory. Playback, pause, stop.
Volume, pitch, panning. Simple streaming for music. Basic mixing.

**Does NOT own:**
Audio file decoding beyond basic WAV/OGG (OGG via bundled stb_vorbis).
DSP effects. Spatial 3D audio. MIDI.

**Types:**
```c
typedef u32 SumiSoundID;
typedef u32 SumiVoiceID;
#define SUMI_INVALID_SOUND 0
#define SUMI_INVALID_VOICE 0

typedef struct SumiAudioDesc {
    const u8*  data;        // raw PCM or OGG; Sumi copies
    usize      data_size;
    i32        channels;    // 1 or 2; ignored for OGG (auto-detected)
    i32        sample_rate; // ignored for OGG
    b8         is_ogg;
    b8         stream;      // true = decode on-the-fly (for music)
} SumiAudioDesc;

typedef struct SumiVoiceParams {
    f32  volume;        // 0..1, default 1
    f32  pitch;         // 0.5..2, default 1
    f32  pan;           // -1 (left) .. 1 (right), default 0
    b8   loop;
} SumiVoiceParams;
```

**Public API:**
```c
b8            sumi_audio_init(i32 sample_rate, i32 buffer_frames);
void          sumi_audio_shutdown(void);

SumiSoundID   sumi_sound_create(const SumiAudioDesc* desc);
void          sumi_sound_destroy(SumiSoundID id);

SumiVoiceID   sumi_audio_play(SumiSoundID id, const SumiVoiceParams* params);
void          sumi_audio_stop(SumiVoiceID voice);
void          sumi_audio_pause(SumiVoiceID voice);
void          sumi_audio_resume(SumiVoiceID voice);
void          sumi_audio_set_volume(SumiVoiceID voice, f32 volume);
void          sumi_audio_set_pitch(SumiVoiceID voice, f32 pitch);
void          sumi_audio_set_pan(SumiVoiceID voice, f32 pan);
b8            sumi_audio_is_playing(SumiVoiceID voice);
void          sumi_audio_set_master_volume(f32 volume);
void          sumi_audio_stop_all(void);

// Music (streaming voice, convenience wrappers)
SumiVoiceID   sumi_music_play(SumiSoundID id, f32 volume);
void          sumi_music_stop(SumiVoiceID music);
void          sumi_music_set_volume(SumiVoiceID music, f32 volume);
```

**Implementation notes:**
- Audio runs a platform mixing thread (Win32 WASAPI / CoreAudio / ALSA)
- Voice table: fixed-size array; max voices = `SUMI_MAX_AUDIO_VOICES` (default 64)
- Finished voices are automatically freed; voice IDs become invalid after sound ends
- Streaming sounds decode on the mixing thread; a ring buffer of decoded frames is
  maintained per streaming voice
- `params = NULL` uses defaults (volume 1, pitch 1, pan 0, no loop)

---

### `sumi_collision.h` — 2D Collision

**Responsibility:**
Collision shape definitions. Overlap tests. Point containment. Raycasting.
Collision manifold (penetration depth and normal).

**Does NOT own:**
Physics simulation. Rigid body dynamics. Collision broadphase.
Spatial partitioning (that is your engine layer).

**Types:**
```c
typedef enum SumiShapeType {
    SUMI_SHAPE_AABB,
    SUMI_SHAPE_CIRCLE,
    SUMI_SHAPE_SEGMENT,    // line segment (for raycasting)
    SUMI_SHAPE_POLYGON,    // convex polygon, max SUMI_MAX_POLY_VERTS verts
} SumiShapeType;

#define SUMI_MAX_POLY_VERTS 16

typedef struct SumiShape {
    SumiShapeType type;
    union {
        struct { SumiVec2 min, max; }                        aabb;
        struct { SumiVec2 center; f32 radius; }              circle;
        struct { SumiVec2 a, b; }                            segment;
        struct { SumiVec2 verts[SUMI_MAX_POLY_VERTS];
                 i32 count; }                                polygon;
    };
} SumiShape;

typedef struct SumiManifold {
    b8       hit;
    SumiVec2 normal;          // points from B into A
    f32      depth;           // penetration depth
    SumiVec2 contact;         // approximate contact point
} SumiManifold;

typedef struct SumiRayHit {
    b8       hit;
    f32      t;               // parametric distance along ray [0..1]
    SumiVec2 point;
    SumiVec2 normal;
} SumiRayHit;
```

**Public API:**
```c
// Shape constructors
SumiShape  sumi_shape_aabb(SumiRect rect);
SumiShape  sumi_shape_aabb_minmax(SumiVec2 min, SumiVec2 max);
SumiShape  sumi_shape_circle(SumiVec2 center, f32 radius);
SumiShape  sumi_shape_segment(SumiVec2 a, SumiVec2 b);
SumiShape  sumi_shape_polygon(const SumiVec2* verts, i32 count);
SumiShape  sumi_shape_box(SumiVec2 center, SumiVec2 half_size, f32 rotation);

// Overlap tests
b8           sumi_aabb_vs_aabb(SumiRect a, SumiRect b);
b8           sumi_circle_vs_circle(SumiVec2 ac, f32 ar, SumiVec2 bc, f32 br);
b8           sumi_aabb_vs_circle(SumiRect aabb, SumiVec2 center, f32 radius);
b8           sumi_point_in_aabb(SumiVec2 p, SumiRect r);
b8           sumi_point_in_circle(SumiVec2 p, SumiVec2 center, f32 radius);
b8           sumi_point_in_polygon(SumiVec2 p, const SumiVec2* verts, i32 count);
b8           sumi_shapes_overlap(const SumiShape* a, const SumiShape* b);

// Manifold (penetration + normal)
SumiManifold sumi_manifold_aabb_aabb(SumiRect a, SumiRect b);
SumiManifold sumi_manifold_circle_circle(SumiVec2 ac, f32 ar, SumiVec2 bc, f32 br);
SumiManifold sumi_manifold_aabb_circle(SumiRect aabb, SumiVec2 center, f32 radius);
SumiManifold sumi_manifold_shapes(const SumiShape* a, const SumiShape* b);

// Raycast
SumiRayHit   sumi_ray_vs_aabb(SumiVec2 origin, SumiVec2 dir, f32 max_t, SumiRect rect);
SumiRayHit   sumi_ray_vs_circle(SumiVec2 origin, SumiVec2 dir, f32 max_t,
                                  SumiVec2 center, f32 radius);
SumiRayHit   sumi_ray_vs_segment(SumiVec2 origin, SumiVec2 dir, f32 max_t,
                                   SumiVec2 seg_a, SumiVec2 seg_b);
SumiRayHit   sumi_ray_vs_shape(SumiVec2 origin, SumiVec2 dir, f32 max_t,
                                 const SumiShape* shape);

// Closest points / distances
f32          sumi_dist_point_to_segment(SumiVec2 p, SumiVec2 a, SumiVec2 b);
SumiVec2     sumi_closest_point_on_segment(SumiVec2 p, SumiVec2 a, SumiVec2 b);
SumiVec2     sumi_closest_point_on_aabb(SumiVec2 p, SumiRect r);
```

---

### `sumi_debug.h` — Debug Utilities

**Responsibility:**
Structured logging. Debug assertions. Debug draw overlay (drawn on top of the
game). Profiling zone timing (named sections with microsecond timestamps).

**Does NOT own:**
Production-grade profilers. Remote debug servers. Memory leak detection (use
your allocator's tracking for that).

**Types:**
```c
typedef enum SumiLogLevel {
    SUMI_LOG_TRACE = 0,
    SUMI_LOG_DEBUG,
    SUMI_LOG_INFO,
    SUMI_LOG_WARN,
    SUMI_LOG_ERROR,
    SUMI_LOG_FATAL,
} SumiLogLevel;

typedef void (*SumiLogSinkFn)(SumiLogLevel level, const char* file,
                               i32 line, const char* msg, void* ctx);
```

**Public API:**
```c
// Logging
void  sumi_log_init(SumiLogLevel min_level, SumiLogSinkFn sink, void* ctx);
void  sumi_log_shutdown(void);
void  sumi_log(SumiLogLevel level, const char* file, i32 line, const char* fmt, ...);

#define SUMI_LOG_TRACE(fmt, ...) sumi_log(SUMI_LOG_TRACE, __FILE__, __LINE__, fmt, ##__VA_ARGS__)
#define SUMI_LOG_DEBUG(fmt, ...) sumi_log(SUMI_LOG_DEBUG, __FILE__, __LINE__, fmt, ##__VA_ARGS__)
#define SUMI_LOG_INFO(fmt, ...)  sumi_log(SUMI_LOG_INFO,  __FILE__, __LINE__, fmt, ##__VA_ARGS__)
#define SUMI_LOG_WARN(fmt, ...)  sumi_log(SUMI_LOG_WARN,  __FILE__, __LINE__, fmt, ##__VA_ARGS__)
#define SUMI_LOG_ERROR(fmt, ...) sumi_log(SUMI_LOG_ERROR, __FILE__, __LINE__, fmt, ##__VA_ARGS__)

// Assertions
#ifdef SUMI_DEBUG
  #define SUMI_ASSERT(cond)          do { if (!(cond)) { SUMI_LOG_ERROR("Assert failed: " #cond); __debugbreak(); } } while(0)
  #define SUMI_ASSERT_MSG(cond, msg) do { if (!(cond)) { SUMI_LOG_ERROR("Assert failed: " #cond ": " msg); __debugbreak(); } } while(0)
#else
  #define SUMI_ASSERT(cond)          ((void)0)
  #define SUMI_ASSERT_MSG(cond, msg) ((void)0)
#endif

// Debug draw (rendered after sumi_gfx_flush, before swap)
void  sumi_debug_draw_rect(SumiRect r, SumiColor color);
void  sumi_debug_draw_circle(SumiVec2 center, f32 radius, SumiColor color);
void  sumi_debug_draw_line(SumiVec2 a, SumiVec2 b, SumiColor color);
void  sumi_debug_draw_text(SumiVec2 pos, SumiColor color, const char* fmt, ...);
void  sumi_debug_draw_shape(const SumiShape* shape, SumiColor color);
void  sumi_debug_flush(void);    // call at end of frame to submit debug draws

// Profiling zones
void  sumi_prof_begin(const char* name);
void  sumi_prof_end(void);
void  sumi_prof_frame_end(void);        // call after frame; records frame stats
void  sumi_prof_dump(void);             // print last frame's zone times to log

#define SUMI_PROF_SCOPE(name) \
    sumi_prof_begin(name); \
    for (i32 _pscope = 1; _pscope; _pscope = 0, sumi_prof_end())
```

**Implementation notes:**
- Default log sink writes to stderr (colored via ANSI codes on Unix, SetConsoleTextAttribute on Win32)
- Debug draw uses its own internal batch, separate from the game's batch
- Profiling uses a stack of named zones with `sumi_platform_time_now()` timestamps
- Max profiling zone depth: 32 (configurable via `SUMI_PROF_MAX_DEPTH`)
- All debug functions are no-ops when `SUMI_DEBUG` is not defined

---

## API Cheatsheet

A compact reference grouped by module.

```
═══════════════════════════════════════════════════════════════════════════════
 SUMI FRAMEWORK  —  API CHEATSHEET                              v1.0
═══════════════════════════════════════════════════════════════════════════════

 PLATFORM                                         sumi_platform.h
─────────────────────────────────────────────────────────────────────────────
 sumi_platform_init(&desc)          → b8           init window + GL context
 sumi_platform_poll_events()        → b8           false = quit
 sumi_platform_swap_buffers()                      swap front/back buffer
 sumi_platform_shutdown()                          destroy window
 sumi_platform_get_window_state()   → SumiWindowState
 sumi_platform_set_title(title)
 sumi_platform_time_now()           → u64          nanoseconds
 sumi_platform_clipboard_get()      → const char*
 sumi_platform_clipboard_set(text)
 sumi_platform_show_cursor(b8)
 sumi_platform_lock_cursor(b8)

 TIME                                              sumi_time.h
─────────────────────────────────────────────────────────────────────────────
 sumi_time_init(fixed_hz)
 sumi_time_update()                               call once per frame
 sumi_time_fixed_step()             → b8           consume one fixed step
 sumi_time_delta()                  → f64          frame dt (seconds)
 sumi_time_elapsed()                → f64          total seconds
 sumi_time_frame()                  → u64          frame counter
 sumi_time_get()                    → const SumiTimeState*
 sumi_time_set_fps_cap(fps)
 sumi_time_set_max_delta(max_dt)

 INPUT                                             sumi_input.h
─────────────────────────────────────────────────────────────────────────────
 sumi_input_update()                              call after poll_events
 sumi_input_key_down(key)           → b8
 sumi_input_key_pressed(key)        → b8           one frame only
 sumi_input_key_released(key)       → b8           one frame only
 sumi_input_key_mod(mod)            → b8
 sumi_input_mouse_pos()             → SumiVec2
 sumi_input_mouse_delta()           → SumiVec2
 sumi_input_mouse_scroll()          → f32
 sumi_input_mouse_down(btn)         → b8
 sumi_input_mouse_pressed(btn)      → b8
 sumi_input_mouse_released(btn)     → b8
 sumi_input_text_next(&cp)          → b8           iterate UTF-32 chars
 sumi_input_gamepad(index)          → const SumiGamepad*
 sumi_input_gamepad_button_down(i, btn) → b8

 MEMORY                                           sumi_mem.h
─────────────────────────────────────────────────────────────────────────────
 sumi_arena_create(size, alloc)     → SumiArena
 sumi_arena_destroy(&arena)
 sumi_arena_push(&arena, size)      → void*
 sumi_arena_push_zero(&arena, size) → void*
 sumi_arena_reset(&arena)
 sumi_arena_save(&arena)            → usize        checkpoint
 sumi_arena_restore(&arena, cp)

 sumi_pool_create(elem_sz, cap, a)  → SumiPool
 sumi_pool_destroy(&pool)
 sumi_pool_alloc(&pool)             → void*
 sumi_pool_free(&pool, ptr)
 sumi_pool_reset(&pool)

 sumi_scratch_init(size_each)
 sumi_scratch_push(size)            → void*
 sumi_scratch_reset()               call end of frame

 SUMI_PUSH_STRUCT(arena, T)         → T*
 SUMI_PUSH_ARRAY(arena, T, n)       → T*

 MATH                                             sumi_math.h
─────────────────────────────────────────────────────────────────────────────
 sumi_v2(x,y)                       → SumiVec2
 sumi_v2_add/sub/mul/div(a,b/s)     → SumiVec2
 sumi_v2_dot/cross(a,b)             → f32
 sumi_v2_len/len2/norm(v)           → f32/SumiVec2
 sumi_v2_lerp(a,b,t)                → SumiVec2
 sumi_v2_rotate(v, angle)           → SumiVec2
 sumi_v2_perp(v)                    → SumiVec2    (-y, x)
 sumi_v2_dist(a,b)                  → f32
 sumi_v2_angle(v)                   → f32

 sumi_rect(x,y,w,h)                 → SumiRect
 sumi_rect_center/size(r)           → SumiVec2
 sumi_rect_contains_point(r,p)      → b8
 sumi_rect_overlaps(a,b)            → b8
 sumi_rect_intersect/union(a,b)     → SumiRect
 sumi_rect_expand/offset(r,...)     → SumiRect

 sumi_mat3_identity()               → SumiMat3
 sumi_mat3_translate/rotate/scale() → SumiMat3
 sumi_mat3_mul(a,b)                 → SumiMat3
 sumi_mat3_transform_point(m,p)     → SumiVec2
 sumi_mat3_inverse(m)               → SumiMat3

 sumi_lerp/clamp/clamp01(...)       → f32
 sumi_smoothstep(e0,e1,x)           → f32
 sumi_remap(v,il,ih,ol,oh)          → f32
 sumi_snap(v, grid)                 → f32

 COLOR                                            sumi_color.h
─────────────────────────────────────────────────────────────────────────────
 sumi_color(r,g,b,a)                → SumiColor
 sumi_color_hex(0xRRGGBBAA)         → SumiColor
 sumi_color_hsv(h,s,v)             → SumiColor
 sumi_color_lerp(a,b,t)            → SumiColor
 sumi_color_fade(c, alpha)          → SumiColor
 sumi_color_tint(c, tint)           → SumiColor
 sumi_color_to_float(c)             → SumiColorF
 SUMI_WHITE / BLACK / RED / GREEN / BLUE / BLANK ...

 DYNAMIC ARRAY                                    sumi_buf.h
─────────────────────────────────────────────────────────────────────────────
 sumi_buf_push(buf, item)
 sumi_buf_pop(buf)
 sumi_buf_count(buf)                → usize
 sumi_buf_reserve(buf, n)
 sumi_buf_clear(buf)
 sumi_buf_free(buf)
 sumi_buf_remove_swap(buf, i)

 IMAGE                                            sumi_image.h
─────────────────────────────────────────────────────────────────────────────
 sumi_image_decode(data, size)      → SumiImage    auto-detect
 sumi_image_decode_png/bmp/qoi(...) → SumiImage
 sumi_image_encode_png/qoi(img, &sz)→ u8*          caller frees
 sumi_image_create(w,h,fmt)         → SumiImage
 sumi_image_free(&img)
 sumi_image_get_pixel(img,x,y)      → SumiColor
 sumi_image_set_pixel(img,x,y,c)
 sumi_image_flip_v/h(&img)
 sumi_image_crop(&img, rect)        → SumiImage

 ASSETS                                           sumi_asset.h
─────────────────────────────────────────────────────────────────────────────
 sumi_asset_read_file(path)         → SumiFileData
 sumi_asset_free(&fd)
 sumi_asset_write_file(path,d,sz)   → b8
 sumi_asset_file_exists(path)       → b8
 sumi_vfs_init()
 sumi_vfs_mount(virt, real)
 sumi_vfs_read(virt_path)           → SumiFileData

 TEXTURE                                          sumi_texture.h
─────────────────────────────────────────────────────────────────────────────
 sumi_texture_create(&desc)         → SumiTextureID   0 = fail
 sumi_texture_from_image(&img, flt) → SumiTextureID
 sumi_texture_create_blank(w,h,c)   → SumiTextureID
 sumi_texture_destroy(id)
 sumi_texture_update(id,data,x,y,w,h)
 sumi_texture_get_info(id)          → SumiTextureInfo
 sumi_subtexture(id, rect)          → SumiSubTexture

 sumi_atlas_create(w,h,fmt,alloc)   → SumiAtlas*
 sumi_atlas_destroy(atlas)
 sumi_atlas_pack(atlas, &img, name) → SumiSubTexture
 sumi_atlas_find(atlas, name)       → SumiSubTexture
 sumi_atlas_get_texture(atlas)      → SumiTextureID

 SHADER                                           sumi_shader.h
─────────────────────────────────────────────────────────────────────────────
 sumi_shader_create(vert,frag)      → SumiShaderID
 sumi_shader_destroy(id)
 sumi_shader_set_int/float/vec2/vec4/mat3/mat4(id,name,val)
 sumi_shader_set_texture_slot(id,name,slot)
 sumi_shader_default()              → SumiShaderID   sprite batch
 sumi_shader_sdf()                  → SumiShaderID   SDF text
 sumi_shader_screen()               → SumiShaderID   post-process base

 GRAPHICS                                         sumi_gfx.h
─────────────────────────────────────────────────────────────────────────────
 sumi_gfx_init(&config)             → b8
 sumi_gfx_shutdown()
 sumi_gfx_begin_frame()
 sumi_gfx_end_frame()               implicit flush
 sumi_gfx_flush()                   explicit flush
 sumi_gfx_clear(color)
 sumi_gfx_set_viewport(x,y,w,h)
 sumi_gfx_set_projection(mat4)
 sumi_gfx_set_blend_mode(mode)
 sumi_gfx_set_shader(id)
 sumi_gfx_set_scissor(rect)
 sumi_gfx_clear_scissor()

 sumi_render_target_create(w,h,fmt) → SumiRenderTargetID
 sumi_render_target_destroy(id)
 sumi_gfx_set_render_target(id)     0 = screen
 sumi_render_target_get_texture(id) → SumiTextureID

 sumi_gfx_draw_texture(tex,pos,tint)
 sumi_gfx_draw_subtexture(sub,pos,tint)
 sumi_gfx_draw_sprite(tex,src,dst,origin,rot,tint)
 sumi_gfx_draw_rect_fill(rect,color)
 sumi_gfx_draw_rect_outline(rect,thickness,color)
 sumi_gfx_draw_circle_fill(center,r,segs,color)
 sumi_gfx_draw_line(a,b,thickness,color)
 sumi_gfx_draw_triangle(a,b,c,color)
 sumi_gfx_draw_polygon(verts,n,color)
 sumi_gfx_get_stats()               → SumiGfxStats

 FONT                                             sumi_font.h
─────────────────────────────────────────────────────────────────────────────
 sumi_font_create(&desc)            → SumiFontID
 sumi_font_destroy(id)
 sumi_font_measure(id, text, px)    → SumiTextMetrics
 sumi_font_draw(id, text, pos, px, color)
 sumi_font_draw_wrapped(id, text, bounds, px, color)
 sumi_font_draw_aligned(id, text, pos, px, color, ax, ay)

 CAMERA                                           sumi_camera.h
─────────────────────────────────────────────────────────────────────────────
 sumi_camera2d_default()            → SumiCamera2D
 sumi_camera2d_get_projection(&cam,w,h) → SumiMat4
 sumi_camera2d_world_to_screen(&cam,p,w,h) → SumiVec2
 sumi_camera2d_screen_to_world(&cam,p,w,h) → SumiVec2
 sumi_camera2d_get_world_bounds(&cam,w,h)  → SumiRect
 sumi_camera2d_follow(&cam, target, speed, dt)

 sumi_camera_shake_create(mag,freq,dur) → SumiCameraShake
 sumi_camera_shake_update(&shake,&cam,dt)

 AUDIO                                            sumi_audio.h
─────────────────────────────────────────────────────────────────────────────
 sumi_audio_init(sample_rate, frames) → b8
 sumi_audio_shutdown()
 sumi_sound_create(&desc)           → SumiSoundID
 sumi_sound_destroy(id)
 sumi_audio_play(id, &params)       → SumiVoiceID
 sumi_audio_stop/pause/resume(voice)
 sumi_audio_set_volume/pitch/pan(voice, val)
 sumi_audio_is_playing(voice)       → b8
 sumi_audio_set_master_volume(vol)
 sumi_audio_stop_all()
 sumi_music_play(id, vol)           → SumiVoiceID

 COLLISION                                        sumi_collision.h
─────────────────────────────────────────────────────────────────────────────
 sumi_shape_aabb(rect)              → SumiShape
 sumi_shape_circle(center,r)        → SumiShape
 sumi_shape_polygon(verts,n)        → SumiShape

 sumi_aabb_vs_aabb(a,b)             → b8
 sumi_circle_vs_circle(ac,ar,bc,br) → b8
 sumi_aabb_vs_circle(rect,c,r)      → b8
 sumi_point_in_aabb(p,r)            → b8
 sumi_point_in_circle(p,c,r)        → b8
 sumi_shapes_overlap(&a,&b)         → b8

 sumi_manifold_aabb_aabb(a,b)       → SumiManifold
 sumi_manifold_aabb_circle(r,c,r)   → SumiManifold
 sumi_manifold_shapes(&a,&b)        → SumiManifold

 sumi_ray_vs_aabb(orig,dir,t,rect)  → SumiRayHit
 sumi_ray_vs_circle(orig,dir,t,c,r) → SumiRayHit
 sumi_ray_vs_shape(orig,dir,t,&sh)  → SumiRayHit

 DEBUG                                            sumi_debug.h
─────────────────────────────────────────────────────────────────────────────
 SUMI_LOG_INFO/WARN/ERROR(fmt, ...)
 SUMI_ASSERT(cond)
 SUMI_ASSERT_MSG(cond, msg)

 sumi_debug_draw_rect(rect, color)
 sumi_debug_draw_circle(center,r,color)
 sumi_debug_draw_line(a,b,color)
 sumi_debug_draw_text(pos, color, fmt, ...)
 sumi_debug_draw_shape(&shape, color)
 sumi_debug_flush()

 SUMI_PROF_SCOPE("name") { ... }
 sumi_prof_begin("name")
 sumi_prof_end()
 sumi_prof_frame_end()
 sumi_prof_dump()
═══════════════════════════════════════════════════════════════════════════════
```

---

## Usage Examples

### Minimal Window + Clear

```c
#define SUMI_IMPLEMENTATION
#include "sumi.h"

int main(void) {
    SumiWindowDesc win = {
        .title  = "Sumi Window",
        .width  = 1280,
        .height = 720,
        .vsync  = 1,
    };
    sumi_platform_init(&win);
    sumi_gfx_init(NULL);   // default config
    sumi_time_init(60.0);
    sumi_input_update();

    while (sumi_platform_poll_events()) {
        sumi_time_update();
        sumi_input_update();

        sumi_gfx_begin_frame();
        sumi_gfx_clear(SUMI_BLACK);
        sumi_gfx_end_frame();

        sumi_platform_swap_buffers();
    }

    sumi_gfx_shutdown();
    sumi_platform_shutdown();
    return 0;
}
```

---

### Sprite Rendering with Camera

```c
SumiCamera2D cam = sumi_camera2d_default();
SumiWindowState ws = sumi_platform_get_window_state();
cam.offset = sumi_v2(ws.width * 0.5f, ws.height * 0.5f);
cam.zoom   = 2.0f;

// Load texture
SumiFileData fd  = sumi_vfs_read("/assets/player.png");
SumiImage    img = sumi_image_decode(fd.data, fd.size);
sumi_asset_free(&fd);
SumiTextureID tex = sumi_texture_from_image(&img, SUMI_FILTER_NEAREST);
sumi_image_free(&img);

// Frame
sumi_gfx_begin_frame();
sumi_gfx_clear(sumi_color_hex(0x222034FF));

SumiMat4 proj = sumi_camera2d_get_projection(&cam,
                    (f32)ws.width, (f32)ws.height);
sumi_gfx_set_projection(proj);

SumiRect src = sumi_rect(0, 0, 16, 16);     // sprite sheet source
SumiRect dst = sumi_rect(player.x, player.y, 32, 32);
SumiVec2 origin = sumi_v2(16, 16);          // center pivot
sumi_gfx_draw_sprite(tex, src, dst, origin, player.rotation, SUMI_WHITE);

sumi_gfx_end_frame();
sumi_platform_swap_buffers();
```

---

### Fixed Timestep Game Loop

```c
sumi_time_init(60.0);   // 60Hz fixed update

while (sumi_platform_poll_events()) {
    sumi_time_update();
    sumi_input_update();

    // Fixed update (physics, game logic)
    while (sumi_time_fixed_step()) {
        f64 dt = sumi_time_get()->fixed_delta;
        physics_update(dt);
        entity_update(dt);
    }

    // Variable update (animation, interpolation)
    f64 dt = sumi_time_delta();
    animation_update(dt);

    // Render
    sumi_gfx_begin_frame();
    sumi_gfx_clear(SUMI_BLACK);
    render_world();
    sumi_gfx_end_frame();
    sumi_platform_swap_buffers();
}
```

---

### Event System

```c
// Register event types at startup
SumiEventType EV_ENEMY_KILLED = sumi_event_register("enemy_killed");
SumiEventType EV_ITEM_PICKUP  = sumi_event_register("item_pickup");

// Listener
typedef struct { i32 enemy_id; i32 score; } EnemyKilledData;
void on_enemy_killed(const SumiEvent* e, void* ctx) {
    EnemyKilledData* d = (EnemyKilledData*)e->data;
    score_system_add(d->score);
    fx_spawn_death(d->enemy_id);
}
sumi_event_listen(EV_ENEMY_KILLED, on_enemy_killed, NULL);

// Dispatch in game loop
while (...) {
    sumi_time_update();
    sumi_input_update();

    game_update(sumi_time_delta());     // game logic pushes events

    sumi_event_dispatch();              // listeners called here

    game_render();
    sumi_platform_swap_buffers();
}

// Somewhere in game logic
EnemyKilledData payload = { .enemy_id = id, .score = 100 };
sumi_event_push(EV_ENEMY_KILLED, &payload, sizeof(payload));
```

---

### Collision Response

```c
SumiRect player_rect = sumi_rect(player.x, player.y, 16, 16);

for (i32 i = 0; i < enemy_count; i++) {
    SumiRect enemy_rect = sumi_rect(enemies[i].x, enemies[i].y, 12, 12);
    SumiManifold m = sumi_manifold_aabb_aabb(player_rect, enemy_rect);
    if (m.hit) {
        // Separate player by penetration
        player.x += m.normal.x * m.depth;
        player.y += m.normal.y * m.depth;
        player.health -= enemies[i].damage;
    }
}
```

---

### Render Target (Post-Processing)

```c
// Create offscreen target
SumiRenderTargetID rt = sumi_render_target_create(1280, 720, SUMI_PIXEL_RGBA8);
SumiTextureID      rt_tex = sumi_render_target_get_texture(rt);
SumiShaderID       crt_shader = sumi_shader_create(vert_src, crt_frag_src);

// Each frame
sumi_gfx_begin_frame();

// Pass 1: render game to offscreen buffer
sumi_gfx_set_render_target(rt);
sumi_gfx_clear(SUMI_BLACK);
render_game();
sumi_gfx_flush();

// Pass 2: render offscreen buffer to screen with CRT shader
sumi_gfx_set_render_target(0);
sumi_gfx_set_shader(crt_shader);
sumi_shader_set_float(crt_shader, "u_time", (f32)sumi_time_elapsed());
sumi_gfx_draw_texture(rt_tex, sumi_v2(0, 0), SUMI_WHITE);
sumi_gfx_flush();

sumi_gfx_end_frame();
sumi_platform_swap_buffers();
```

---

### Memory — Arena Pattern

```c
// Per-frame temporary string formatting
void* render_frame(SumiArena* frame_arena) {
    usize cp = sumi_arena_save(frame_arena);

    // Build temp string
    char* label = sumi_arena_push(frame_arena, 64);
    snprintf(label, 64, "Score: %d", score);
    sumi_font_draw(font, label, pos, 16, SUMI_WHITE);

    sumi_arena_restore(frame_arena, cp);
    // label is invalid past this point
}

// Or use scratch directly for simple cases
char* tmp = sumi_scratch_push(64);
snprintf(tmp, 64, "FPS: %.0f", fps);
// tmp is valid until sumi_scratch_reset() at frame end
```

---

## Implementation Notes

### Header guard pattern

Each module follows this structure:
```c
#ifndef SUMI_GFX_H
#define SUMI_GFX_H

// === Public API ===
// ... types, function declarations ...

#ifdef SUMI_GFX_IMPLEMENTATION
// === Implementation ===
// ... static functions, internal state, function bodies ...

#ifdef SUMI_TEST
// === Unit Tests ===
// ... test_* functions ...
#endif // SUMI_TEST

#endif // SUMI_GFX_IMPLEMENTATION
#endif // SUMI_GFX_H
```

### Internal state pattern

Each stateful module holds its state in one static struct:
```c
// Internal — not exposed in header
typedef struct {
    // all module state here
    b8 initialized;
} _SumiGfxState;
static _SumiGfxState _gfx = {0};

b8 sumi_gfx_init(const SumiGfxConfig* config) {
    SUMI_ASSERT(!_gfx.initialized);
    // ...
    _gfx.initialized = 1;
    return 1;
}
```

This makes the state easy to reset, inspect in a debugger, and ensures only
one instance of each subsystem exists.

### Dependency compile order

When modules depend on types from other modules:
- `sumi_platform.h` defines primitive typedefs — must be included first
- `sumi_math.h` must be included before `sumi_gfx.h`, `sumi_camera.h`, `sumi_collision.h`
- `sumi_color.h` must be included before `sumi_gfx.h`
- `sumi_texture.h` must be included before `sumi_gfx.h` and `sumi_font.h`
- `sumi_shader.h` must be included before `sumi_gfx.h`
- `sumi_image.h` is independent of GPU modules (intentionally)
- `sumi_asset.h` is independent of GPU modules (intentionally)

The amalgamation `sumi.h` handles this automatically.

### Zero-is-invalid handle convention

Every resource handle type uses `0` as the invalid sentinel. This means
`= {0}` on any struct containing handles produces safe, detectable invalid state.
`SUMI_INVALID_ID`, `SUMI_INVALID_TEXTURE`, `SUMI_INVALID_SHADER`, etc. are all 0.

### Backend abstraction

Internal backend functions follow this naming:
```c
// sumi_gfx_backend.h (internal, not public)
typedef struct _SumiGfxBackend {
    b8   (*init)(void);
    void (*shutdown)(void);
    void (*upload_texture)(u32* handle, const SumiTextureDesc* desc);
    void (*delete_texture)(u32 handle);
    void (*flush_batch)(const _SumiVertex* verts, i32 count, /* ... */);
    // ...
} _SumiGfxBackend;

// Selected at compile time
#ifdef SUMI_BACKEND_OPENGL
  static const _SumiGfxBackend _backend = { ... };
#elif defined(SUMI_BACKEND_METAL)
  static const _SumiGfxBackend _backend = { ... };
#endif
```

The public API in `sumi_gfx.h` never references backend types. All GL/Metal
calls stay behind this boundary.

### Stretchy buffer macro implementation

The dynamic array macros store metadata before the data pointer:
```c
typedef struct { u32 count; u32 capacity; } _SumiBufHeader;
#define _sumi_buf_hdr(buf)   ((_SumiBufHeader*)(buf) - 1)
#define sumi_buf_count(buf)  ((buf) ? _sumi_buf_hdr(buf)->count : 0)
#define sumi_buf_push(buf, item) \
    (_sumi_buf_grow_if_needed((void**)&(buf), sizeof(*(buf))), \
     (buf)[_sumi_buf_hdr(buf)->count++] = (item))
```

### Platform-specific notes

**Windows:** Avoid `windows.h` include bloat — use `#define WIN32_LEAN_AND_MEAN`
and `#define NOMINMAX`. Link `opengl32.lib gdi32.lib winmm.lib kernel32.lib`.

**macOS:** Compile platform file as Objective-C (`-x objective-c`) or as a
separate `.m` file. The rest of Sumi is pure C.

**Linux:** X11 is the default. Compile with `-lGL -lX11 -lXi -lasound -lm`.
Wayland support requires `xdg-shell` protocol and `libwayland-client`.

### Performance expectations

| Operation | Expected cost |
|-----------|---------------|
| `sumi_gfx_draw_sprite()` | ~5ns (CPU batch push) |
| `sumi_gfx_flush()` | 1 draw call (best case) |
| `sumi_input_key_pressed()` | 1 bit test |
| `sumi_arena_push()` | 1 addition + alignment |
| `sumi_v2_add()` | 2 float adds (inlined) |
| `sumi_aabb_vs_aabb()` | 4 comparisons |
| `sumi_strmap_get()` | ~O(1) amortized |

These are indicative, not guaranteed. Profile on your target hardware.

---

*Sumi Framework — build the foundation once, build games forever.*
