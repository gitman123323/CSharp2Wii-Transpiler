# WiiSharp 🎮

**Write Wii homebrew in C#.**

WiiSharp is a C# → C transpiler that lets you write Wii homebrew games and applications using familiar C# syntax, with full IntelliSense support in Visual Studio. No C knowledge required.

---

## What is this?

Instead of writing raw C with libogc, you write clean C# like this:
```csharp
WiiSys.InitVideo();
WiiSys.InitWiimote();

int x = 20;
int y = 12;

while (true)
{
    WiiSys.ScanPads();
    WiiSys.ClearScreen();

    if (WiiSys.IsButtonHeld(WiiButton.Up))    y -= 1;
    if (WiiSys.IsButtonHeld(WiiButton.Down))  y += 1;
    if (WiiSys.IsButtonHeld(WiiButton.Left))  x -= 1;
    if (WiiSys.IsButtonHeld(WiiButton.Right)) x += 1;

    WiiSys.PrintAt(x, y, "@");

    if (WiiSys.ButtonPressed(WiiButton.Home))
        WiiSys.Exit();

    WiiSys.WaitVSync();
}
```

WiiSharp transpiles it to native C, you compile the C file with devkitPPC, and then it produces a `.dol` file that you can run on real Wii hardware at full native speed without ever touching C code at all.

---

## Features

**Language support:**
- Classes, structs, enums, namespaces
- Inheritance with flattened fields and method wrappers
- Arrays with `.Length` support
- String interpolation `$"{}"`
- String comparison and concatenation
- Compound operators `+=` `-=` `*=` `/=`
- Cast expressions `(int)` `(float)`
- `if` `else if` `else`
- `while` and `for` loops
- `switch` statements with `case` and `default`
- `break` and `continue`
- Constructor parameters
- Multi-file projects with automatic dependency resolution
- Reachability analysis — only transpiles what's actually used

**Wii API (`WiiSys`):**
- Video initialization and display
- Wiimote support for 4 players
- Button press and held detection
- Nunchuk joystick and buttons
- IR pointer (x, y, visible)
- Tilt/accelerometer (pitch, roll, yaw)
- Rumble
- Battery level
- Text rendering with colors and positioning
- Screen clearing
- Random numbers
- DrawBox for UI borders
- PrintCenter for centered text
- Timing functions
- Extension detection (Nunchuk, Classic Controller)
- Math functions (Abs, Min, Max, Sqrt, Floor, Ceil)

---

## Requirements

- **Windows** (for the transpiler)
- **.NET 8 SDK** or later
- **Docker Desktop** with the devkitPPC image
- **Visual Studio 2022** (recommended for IntelliSense)

---

## Project Structure
```
WiiSharp/
├── Transpiler/                    ← WiiSharp transpiler (Visual Studio project)
│   ├── bin/Debug/net8.0-windows/  ← Pre-built WiiSharp.exe lives here
│   └── WiiSysLib/                 ← WiiSysAPI.dll lives here — reference this in your game project
├── Examples/                      ← Example projects
│   ├── WiiSharpTestScene/         ← Full API feature test
│   ├── SharpPongWii/              ← Pong with AI and 2 player mode
│   └── SharpSnakeWii/             ← Classic snake game
├── Docker/                        ← devkitPPC Makefile for building .dol files
└── WiiSysAPI.zip                  ← Full source code for the WiiSys library
```

---

## How to Use

### Step 1 — Set up your game project

Create a new C# console project in Visual Studio and add a reference to `Transpiler/WiiSysLib/WiiSysAPI.dll`. This gives you full IntelliSense for all WiiSys functions.

### Step 2 — Write your game
```csharp
using WiiSysAPI;

namespace MyGame
{
    public class Game
    {
        void Main()
        {
            WiiSys.InitVideo();
            WiiSys.InitWiimote();
            WiiSys.ClearScreen();

            while (true)
            {
                WiiSys.ScanPads();
                WiiSys.ClearScreen();

                WiiSys.PrintAt(1, 12, "Hello Wii!");

                if (WiiSys.ButtonPressed(WiiButton.Home))
                    WiiSys.Exit();

                WiiSys.WaitVSync();
            }
        }
    }
}
```

### Step 3 — Transpile

You do not need to build the transpiler yourself — a pre-built executable is already included at:
```
WiiSharp/Transpiler/bin/Debug/net8.0-windows/WiiSharp.exe
```

Just run `WiiSharp.exe`, select your game project folder when prompted and select an output folder. It will generate `out.c`.

If you want to build the transpiler from source instead, open `Transpiler/WiiSharp.sln` in Visual Studio 2022 and build the project.

### Step 4 — Build

Copy `out.c` to your devkitPPC Docker container's `source/` folder and run:
```bash
make
```

This produces `boot.dol` which you can run on your Wii via the Homebrew Channel.

---

## Customizing the WiiSys Library

If you want to add new API functions, change existing behavior or extend the WiiSys library in any way, the full source code is included in `WiiSysAPI.zip` at the root of the WiiSharp folder. Extract it, open it in Visual Studio, make your changes, rebuild the DLL and replace `Transpiler/WiiSysLib/WiiSysAPI.dll` with your updated version.

This is the recommended way to add support for features like TV audio, SD card access, GX graphics or anything else not yet included in WiiSharp.

---

## WiiSys API Reference

### Core
| Method | Description |
|--------|-------------|
| `WiiSys.InitVideo()` | Initialize video output |
| `WiiSys.InitWiimote()` | Initialize all 4 Wiimote channels |
| `WiiSys.ScanPads()` | Scan for controller input (call every frame) |
| `WiiSys.WaitVSync()` | Wait for vertical sync (call every frame) |
| `WiiSys.Exit()` | Return to Homebrew Channel |

### Display
| Method | Description |
|--------|-------------|
| `WiiSys.ClearScreen()` | Clear the screen |
| `WiiSys.Print(text)` | Print text with newline |
| `WiiSys.PrintAt(x, y, text)` | Print text at position |
| `WiiSys.PrintAtInt(x, y, label, value)` | Print int value at position |
| `WiiSys.PrintAtFloat(x, y, label, value)` | Print float value at position |
| `WiiSys.PrintCenter(y, text)` | Print centered text |
| `WiiSys.SetTextColor(TextColor.Red)` | Set text color |
| `WiiSys.DrawBox(x, y, width, height)` | Draw a box border |

### Input
| Method | Description |
|--------|-------------|
| `WiiSys.ButtonPressed(WiiButton.A)` | Button just pressed this frame |
| `WiiSys.IsButtonHeld(WiiButton.Up)` | Button held down |
| `WiiSys.ButtonPressedNunchuk(NunchukButton.Z)` | Nunchuk button pressed |
| `WiiSys.InitNunchuk()` | Initialize nunchuk |
| `WiiSys.NunchukX(player)` | Nunchuk joystick X axis |
| `WiiSys.NunchukY(player)` | Nunchuk joystick Y axis |
| `WiiSys.InitIR()` | Initialize IR pointer |
| `WiiSys.PointerX(player)` | IR pointer X position |
| `WiiSys.PointerY(player)` | IR pointer Y position |
| `WiiSys.PointerVisible(player)` | IR pointer visible |
| `WiiSys.TiltX(player)` | Accelerometer pitch |
| `WiiSys.TiltY(player)` | Accelerometer roll |
| `WiiSys.TiltZ(player)` | Accelerometer yaw |
| `WiiSys.GetExtension(player)` | Detect connected extension |

### Rumble & Feedback
| Method | Description |
|--------|-------------|
| `WiiSys.RumbleOn(player)` | Turn on rumble |
| `WiiSys.RumbleOff(player)` | Turn off rumble |

### Utility
| Method | Description |
|--------|-------------|
| `WiiSys.Random(min, max)` | Random integer |
| `WiiSys.Delay(ms)` | Delay in milliseconds |
| `WiiSys.GetTime()` | Get time in milliseconds |
| `WiiSys.BatteryLevel(player)` | Wiimote battery level |

### Enums
```csharp
WiiButton.A, B, Home, Up, Down, Left, Right, Plus, Minus, One, Two
NunchukButton.Z, C
TextColor.White, Red, Green, Blue, Yellow, Cyan, Magenta
WiiExtension.None, Nunchuk, Classic
```

---

## Current State & Roadmap

WiiSharp currently provides full access to everything you need to start building real text-based Wii homebrew games and applications — input, display, rumble, IR, tilt, nunchuk, timing, math and more. This is a solid foundation for any 2D game or interactive application.

The following features are **not yet implemented** but are planned or open for community contribution:

- **GX graphics** — drawing pixels, shapes, sprites and images directly to the framebuffer
- **TV audio** — playing sound files through the TV speakers via ASND
- **SD card support** — reading and writing files from an SD card
- **OS operations** — file system access, memory management
- **Motion Plus gyroscope** — currently broken at the libogc level, may be revisited in a future version

Since WiiSharp is fully open source, you are welcome and encouraged to extend it however you like — add new API functions, improve the transpiler, add new language features or implement any of the above. The source code is included and well structured so adding new `WiiSys` functions is straightforward.

More features may be added in future versions. If you build something cool with WiiSharp, feel free to share it!

---

## Known Limitations

- **Motion Plus** — Not supported. The gyroscope data is broken at the libogc level and cannot be fixed without rewriting parts of the library.
- **Wiimote speaker** — Not supported. libogc's speaker implementation is broken and does not produce reliable audio.
- **Single-level inheritance only** — `A : B` works, `A : B : C` does not.
- **No generics** — Use arrays instead of `List<T>`.
- **No exceptions** — `try/catch` is not supported on bare metal Wii.
- **No Linq** — Use loops instead.

---

## Examples

> **Note:** All example projects must be transpiled using `WiiSharp.exe` before they can be built and run on the Wii. Point the transpiler at the example project folder and it will generate the `out.c` file ready for devkitPPC.

### WiiSharp Test Scene
A comprehensive test application that demonstrates every feature of the WiiSharp API including IR pointer, tilt/accelerometer, nunchuk, rumble, battery level, extension detection, colors, DrawBox, random numbers and more. Use this as a reference when building your own projects.

### Sharp Pong Wii
Full Pong implementation with AI opponent or 2 player local multiplayer via Nunchuk. Score tracking, rumble feedback on paddle hits and scoring, win screen.

### Sharp Snake Wii
Classic Snake game with array based growing tail, self collision, random food placement, score tracking, box border UI, speed increase every 5 food and rumble on food collection.

---

## How it works

WiiSharp uses **Roslyn** (the .NET compiler platform) to parse your C# code into a syntax tree, then walks the tree and generates equivalent C code. The generated C is compiled by **devkitPPC** (a GCC-based cross-compiler for PowerPC) into a `.dol` file that runs natively on the Wii's Broadway CPU.

The key insight is that C# and C are structurally similar enough that most constructs translate directly:
- C# classes → C structs + functions
- C# inheritance → flattened C structs
- C# string interpolation → C printf format strings
- C# `bool` → C `int`

The WiiSys API is a thin wrapper around **libogc** — the open source Wii homebrew library.

---

## Credits

Built by **Loghan Veilleux** in a single session using C# and Roslyn.

Special thanks to the **devkitPro** and **libogc** teams for making Wii homebrew development possible.

---

## License

MIT License — free to use, modify and distribute. Credit appreciated but not required.

**Please do not claim this as your own work without credit.**
