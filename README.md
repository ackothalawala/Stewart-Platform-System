[README 2.md](https://github.com/user-attachments/files/27625369/README.2.md)
# Stewart-Platform-System# Stewart Platform V2 — 6-DOF Closed-Loop Motion System

![Platform](https://img.shields.io/badge/Platform-Windows%20%7C%20Linux%20%7C%20Raspberry%20Pi-blue)
![Language](https://img.shields.io/badge/Language-C%23%20%2B%20Arduino%20C%2B%2B-informational)
![Framework](https://img.shields.io/badge/UI-Avalonia%20UI-purple)
![Renderer](https://img.shields.io/badge/3D-Silk.NET%20OpenGL-orange)
![Controller](https://img.shields.io/badge/MCU-ESP32-red)
![Status](https://img.shields.io/badge/Status-Active%20Development-brightgreen)

A 6-degree-of-freedom Stewart Platform with real-time closed-loop orientation control. The ESP32 microcontroller runs a proportional correction loop at 30 Hz using live MPU6050 feedback mounted on the moving platform. A cross-platform C# desktop application built on Avalonia UI handles inverse kinematics computation, 3D visualisation via raw OpenGL (Silk.NET), and bidirectional communication over Wi-Fi WebSocket or USB Serial.

This project was developed as part of undergraduate research in the Information Systems department at Sabaragamuwa University of Sri Lanka.

---

## Research Context

This system is part of the undergraduate research project:

> **"An IoT-Enabled, Software-Defined Cricket Training Platform Using Stewart Platform for Player Monitoring and Skill Development"**
> 
> BSc (Hons) Information Systems — Sabaragamuwa University of Sri Lanka

---

## Demo & Screenshots

### Physical Stewart Platform

![Hardware Prototype](Assets/platform_hardware.jpg)

### Desktop Application — Open Loop Mode

![Open Loop UI](Images/desktop_openloop.png)

### Desktop Application — Closed Loop Mode

![Closed Loop UI](Images/desktop_closedloop.png)

### Real-Time OpenGL 3D Visualiser

![3D Visualiser](Images/3d_visualiser.png)

### Fusion 360 / CAD Design

<p align="center" style="display: flex; justify-content: center; gap: 10px;">
  <img src="Assets/base 3d model 1.png" width="300" alt="Autocad Base 3D Model 1">
  <img src="Assets/base 3d model 2.png" width="300" alt="Autocad Base 3D Model 2">
  <img src="Assets/base 3d model 3.png" width="300" alt="Autocad Base 3D Model 3">
</p>

### Demo Video

[Watch Demo Video](https://youtube.com/your-link)

---

## Repository Structure

```
Stewart-Platform-System/
│
├── Assets/                               # Images, screenshots, diagrams, GIFs
│   ├── hero.jpg
│   ├── platform_hardware.jpg
│   ├── desktop_openloop.png
│   ├── desktop_closedloop.png
│   ├── 3d_visualiser.png
│   ├── fusion_assembly.png
│   ├── system_architecture.png
│   └── demo.gif
│
├── StewartPlatform_WPF/                  # Version 1 — Windows desktop app
│   ├── MainWindow.xaml                   # WPF layout with HelixToolkit viewport
│   ├── MainWindow.xaml.cs                # WPF UI logic
│   ├── StewartMath.cs                    # Inverse kinematics solver
│   └── RobotConfig.cs                    # Physical configuration
│
├── StewartPlatform_Avalonia/             # Version 2 — Cross-platform desktop app
│   ├── MainWindow.axaml                  # Avalonia UI layout
│   ├── MainWindow.axaml.cs               # UI logic, connection, timers
│   ├── PlatformView3D.cs                 # OpenGL renderer (Silk.NET)
│   ├── StewartMath.cs                    # IK solver (System.Numerics)
│   └── RobotConfig.cs                    # Physical dimensions and geometry
│
├── Firmware/
│   └── stewart_avalonia_04.ino           # ESP32 firmware
│
│
└── README.md
```

---

## Project History

### Version 1 — WPF (Windows only)

The first desktop application was built using C# WPF with HelixToolkit for 3D rendering. It provided full open-loop control of the Stewart Platform — X/Y/Z translation, Roll/Pitch/Yaw rotation, live servo angle readout, and MPU6050 sensor feedback display. The 3D view rendered the base ring, moving platform, horns, and rods using HelixToolkit's scene graph.

This version was Windows-exclusive and had no closed-loop control. The MPU6050 data was displayed as monitoring information only — it had no influence on servo commands.

### Version 2 — Avalonia UI (current)

The application was rewritten from scratch in Avalonia UI to target Windows, Linux, and Raspberry Pi from a single codebase. HelixToolkit was replaced by a custom OpenGL renderer built directly on Silk.NET using Avalonia's `OpenGlControlBase`. The firmware was extended to support a closed-loop proportional control mode where the ESP32 closes the orientation loop locally using live MPU6050 feedback, removing network latency from the control path entirely.

| | V1 — WPF | V2 — Avalonia |
|---|---|---|
| UI framework | WPF | Avalonia UI |
| 3D rendering | HelixToolkit | Silk.NET (raw OpenGL) |
| Target platform | Windows only | Windows, Linux, Raspberry Pi |
| Control mode | Open loop only | Open loop + Closed loop |
| Orientation feedback | Display only | Active control input |
| Disturbance rejection | No | Yes |

---

## Features

### Firmware (ESP32 — `stewart_avalonia_04.ino`)

- Wi-Fi Access Point mode (`StewartPlatform_AP`) — no external router required
- WebSocket server on port 81 for primary communication
- USB Serial fallback at 115,200 baud
- MPU6050 integration: Roll and Pitch from accelerometer, Yaw from integrated gyroscope
- Gyroscope offset calibration on boot (100-sample average)
- PCA9685 PWM servo driver via I2C — per-servo calibrated middle pulse widths
- **Open-loop mode**: receives orientation targets pre-computed by the PC IK solver and maps them to servo pulse widths
- **Closed-loop mode**: proportional controller runs at 30 Hz on the ESP32; PC sends only Roll/Pitch/Yaw setpoints; ESP32 reads MPU6050, computes error, and applies servo corrections through an axis-to-servo influence matrix
- Servo correction clamped to ±2° per axis per cycle (±1° for yaw)
- Servo angle clamped to ±25° total from home position
- Feedback broadcast every 100 ms over WebSocket and Serial: `FB:roll,pitch,yaw,temp,mode`
- Smooth mode switching: entering closed-loop seeds the setpoint from the current measured orientation to prevent jumps

### Desktop Application (Avalonia V2)

- Cross-platform: Windows, Linux, Raspberry Pi — single binary
- Real-time 3D visualisation of the platform geometry using raw OpenGL (Silk.NET)
- Cross-platform shader selection: `#version 120` Desktop GL on Linux/Pi, `#version 100` ANGLE/ES on Windows
- Renders base attachment ring (blue), moving platform ring (green), servo horns (yellow), connecting rods (red)
- Stewart Platform inverse kinematics solver — ZYX Euler rotation matrix, per-servo horn geometry, `System.Numerics` float arithmetic
- **Open-loop controls**: X/Y/Z translation ±35 mm, Roll/Pitch/Yaw rotation ±12°
- **Closed-loop controls**: rotation sliders set orientation setpoints; error display per axis; loop status `✓ LOCKED` / `▶ MOVING`; ±0.5° position tolerance
- Animated reset-to-home with cubic ease-out (30 steps at ~60 fps)
- Mode toggle button: `⚙ OPEN LOOP` ↔ `🔒 CLOSED LOOP`
- Connection: Wi-Fi WebSocket, USB Serial, RF Dongle (Arduino Nano)
- 3D view refresh at 30 Hz; sensor display refresh at 10 Hz
- Live servo angle readout for all 6 servos (1 decimal place)

---

## Hardware

| Component | Qty | Purpose |
|---|---|---|
| ESP32 development board | 1 | Main controller, Wi-Fi AP, WebSocket server, PID loop |
| PCA9685 16-ch PWM driver | 1 | I2C-controlled 12-bit PWM output for all 6 servos |
| Servo motors | 6 | Platform actuation — one per leg |
| MPU6050 6-axis IMU | 1 | Real-time orientation feedback — mounted on moving platform |
| Stewart Platform frame | 1 | Machined/printed base, top plate, 6 horns, 6 connecting rods |
| Universal joints | 12 | Multi-axis rotation at rod endpoints |
| Threaded rods + sleeve couplers | 6 sets | Adjustable-length actuator rods |
| 5–6V regulated power supply | 1 | Servo power rail — separate from ESP32 logic supply |

> The MPU6050 must be mounted rigidly on the **top moving platform**, not the base. Any mounting flex or vibration will introduce noise into the orientation measurement and degrade closed-loop performance.

---

## Physical Dimensions

The following values are defined in `RobotConfig.cs` and must accurately reflect your physical build. Incorrect values produce wrong IK solutions and the platform will not move as expected.

| Parameter | Value | Notes |
|---|---|---|
| Base radius | 91.56 mm | Centre to base attachment point |
| Platform radius | 56.00 mm | Centre to platform attachment point |
| Horn length | 36.845 mm | Servo shaft to rod ball joint |
| Rod length | 144.00 mm | Ball joint to ball joint |
| Initial height | 132.89 mm | Home position platform height |
| Home yaw offset | 30° | Geometric correction for attachment layout |
| Max translation | ±30 mm | Software-enforced limit |
| Max rotation | ±30° | Software-enforced limit |

---

## Wiring

### I2C Bus — ESP32 to PCA9685 and MPU6050

Both the PCA9685 and MPU6050 share the same I2C bus. Default I2C addresses are `0x40` (PCA9685) and `0x68` (MPU6050).

| ESP32 Pin | Connected to |
|---|---|
| GPIO 21 (SDA) | PCA9685 SDA, MPU6050 SDA |
| GPIO 22 (SCL) | PCA9685 SCL, MPU6050 SCL |
| 3.3V | PCA9685 VCC, MPU6050 VCC |
| GND | PCA9685 GND, MPU6050 GND |

### PCA9685 to Servos

| PCA9685 Channel | Servo |
|---|---|
| Channel 0 | Servo 0 |
| Channel 1 | Servo 1 |
| Channel 2 | Servo 2 |
| Channel 3 | Servo 3 |
| Channel 4 | Servo 4 |
| Channel 5 | Servo 5 |

> Servo V+ must be supplied from a **dedicated 5–6V rail**. Do not power servos from the ESP32 3.3V pin — the current draw will cause brownouts and reset the ESP32 mid-operation.

---

## Mechanical Design

### Structure

The platform follows the classic Gough-Stewart parallel manipulator layout:

- **Base plate**: fixed, holds 6 servo motor mounts in a roughly hexagonal arrangement
- **Top plate**: the moving platform; carries the MPU6050 IMU
- **Horns**: short lever arms press-fitted to each servo shaft; convert rotary motion to linear push
- **Connecting rods**: adjustable-length assemblies linking horn ball joints to top plate ball joints

### Rod Assembly

Each leg follows this configuration:

```
[Top Plate Ball Joint] — Shaft — [Threaded Sleeve] — Shaft — [Horn Ball Joint]
```

The threaded sleeve allows continuous length adjustment for calibrating the platform's home position without modifying firmware constants.

### Home Yaw Offset

The attachment point layout is geometrically optimised for workspace and stiffness, which introduces a 30° yaw offset at the home position relative to the geometric centroid. This is compensated by `HomeYawOffsetDeg = 30.0` in `RobotConfig.cs`, which rotates the platform body-frame attachment points during IK initialisation.

---

## Kinematics

### Inverse Kinematics (`StewartMath.cs`)

Given a desired 6-DOF pose — translation `(tx, ty, tz)` in mm and rotation `(rx, ry, rz)` in radians — the solver computes the required servo horn angle `α` for each of the 6 actuators independently.

**Algorithm per servo `i`:**

1. Apply ZYX Euler rotation matrix to platform body-frame attachment point `p[i]`:

```
q[i] = R(rz, ry, rx) × p[i] + translation + h0
```

where `h0 = (0, 0, InitialHeight)` is the home height offset.

2. Compute the leg vector from base attachment `b[i]` to rotated platform point `q[i]`:

```
l = q[i] - b[i]
```

3. Solve for horn angle `α` using the standard Stewart Platform IK formulation:

```
L = |l|² - RodLength² + HornLength²
M = 2 × HornLength × (q.z - b.z)
N = 2 × HornLength × (cos(β) × (q.x - b.x) + sin(β) × (q.y - b.y))

α = arcsin(L / √(M² + N²)) - arctan2(N, M)
```

4. Compute horn end-point in world space for 3D rendering:

```
HornEnd = (HornLength × cos(α) × cos(β) + b.x,
           HornLength × cos(α) × sin(β) + b.y,
           HornLength × sin(α) + b.z)
```

`β` is the servo base orientation angle defined in `RobotConfig.BetaAngles`. The result is clamped to `[-1, 1]` before `arcsin` to guard against numerical errors at the workspace boundary.

### Forward Kinematics

Not implemented. Forward kinematics for a Stewart Platform requires solving a nonlinear system of 6 equations simultaneously, which is computationally intensive and has multiple solutions. Since the MPU6050 provides direct orientation measurement, forward kinematics are not needed for this control approach.

---

## System Architecture

<p align="center">
  <img src="Assets/System Architecture.png" width="800" alt="Stewart Platform System Architecture">
</p>

**Data flow — open-loop:**
Slider → IK solver (PC) → servo angles → WebSocket/Serial → `parseCommand` (ESP32) → PCA9685

**Data flow — closed-loop:**
Slider → setpoint → WebSocket/Serial → `TARGET:r,p,y` → ESP32 proportional controller → MPU6050 feedback → servo corrections → PCA9685

---

## Software Dependencies

### Desktop App — Avalonia V2

Install via NuGet Package Manager or `.csproj`:

```xml
<PackageReference Include="Avalonia"                Version="11.*" />
<PackageReference Include="Avalonia.Desktop"         Version="11.*" />
<PackageReference Include="Avalonia.Themes.Fluent"   Version="11.*" />
<PackageReference Include="Silk.NET.OpenGL"          Version="2.*"  />
```

### Desktop App — WPF V1 (Windows only)

```xml
<PackageReference Include="HelixToolkit.Wpf" Version="2.*" />
```

### ESP32 Firmware

Install via **Arduino IDE Library Manager**:

| Library | Author | Purpose |
|---|---|---|
| `Adafruit PWM Servo Driver Library` | Adafruit | PCA9685 I2C control |
| `Adafruit MPU6050` | Adafruit | IMU sensor driver |
| `Adafruit Unified Sensor` | Adafruit | Sensor abstraction layer |
| `WebSockets` | Markus Sattler | WebSocket server on ESP32 |

**ESP32 board package** — add this URL to Arduino IDE → Preferences → Additional Boards Manager URLs:

```
https://raw.githubusercontent.com/espressif/arduino-esp32/gh-pages/package_esp32_index.json
```

Then install: **ESP32 by Espressif Systems** via Boards Manager. Select **ESP32 Dev Module** when uploading.

---

## Getting Started

### Step 1 — Flash the Firmware

1. Open `Firmware/stewart_avalonia_04.ino` in Arduino IDE
2. Select board: **ESP32 Dev Module**
3. Install all firmware libraries listed above
4. Upload the sketch
5. Open Serial Monitor at **115200 baud** — expected output:

```
Stewart Platform Starting...
AP IP: 192.168.4.1
MPU6050 Found - Calibrating...
Calibration Complete!
Ready!
```

If `MPU6050 NOT FOUND!` appears, check your SDA/SCL wiring and I2C address.

### Step 2 — Calibrate Servo Middle Positions

The firmware uses per-servo middle pulse width constants. Adjust these to level all horns at the home position:

```cpp
constexpr uint16_t SERVO_MIDDLE_PULSE_WIDTH[6] = {
    1522-48,   // Servo 0
    1522+60,   // Servo 1
    1522+0,    // Servo 2
    1522+14,   // Servo 3
    1522-28,   // Servo 4
    1522-116   // Servo 5
};
```

Call `resetToZero()` and verify each horn is horizontal before proceeding. An incorrectly levelled horn shifts the platform's home position and introduces asymmetric error into the closed-loop controller.

### Step 3 — Update Physical Dimensions

Edit `RobotConfig.cs` in the Avalonia project to match your build measurements:

```csharp
public double BaseRadius     { get; set; } = 91.56;
public double PlatformRadius { get; set; } = 56.0;
public double HornLength     { get; set; } = 36.845;
public double RodLength      { get; set; } = 144.0;
public double InitialHeight  { get; set; } = 132.890;
public double HomeYawOffsetDeg { get; set; } = 30.0;
```

### Step 4 — Build and Run the Desktop App

**Avalonia V2 (recommended — all platforms):**

```bash
cd StewartPlatform_Avalonia
dotnet build
dotnet run
```

**WPF V1 (Windows only):**

```bash
cd StewartPlatform_WPF
dotnet build
dotnet run
```

### Step 5 — Connect

**Wi-Fi:**
1. Connect your PC to the `StewartPlatform_AP` Wi-Fi network (password: `12345678`)
2. In the app, click **Connect Platform**
3. Select **Wi-Fi (WebSocket)**
4. URL: `ws://192.168.4.1:81`
5. Click **Connect**

**USB Serial:**
1. Connect ESP32 via USB cable
2. Click **Connect Platform**
3. Select **USB Serial**
4. Choose the correct COM port from the list
5. Click **Connect**

After connecting, the app sends `MODE:OPEN` automatically. The status bar turns green and shows the connection type.

---

## Usage

### Open Loop Mode

The default mode on startup. The PC IK solver computes all 6 servo angles from the slider positions and sends them to the ESP32:

- **Position (mm):** X/Y/Z translation sliders move the platform laterally. Range: ±35 mm.
- **Rotation (deg):** Roll/Pitch/Yaw sliders tilt the platform. Range: ±12°.
- Text boxes next to each slider accept direct numeric input — type a value and click **Set**.
- **Reset Pose** animates all axes back to zero using a cubic ease-out curve over ~0.5 seconds.

The 3D view updates at 30 Hz and always reflects the current IK solution, regardless of connection state.

### Switching to Closed Loop

1. Click the **`⚙ OPEN LOOP`** button. It changes to **`🔒 CLOSED LOOP`** (blue).
2. The app sends `MODE:CLOSED` to the ESP32.
3. The ESP32 seeds its internal setpoint from the current measured orientation to prevent a sudden jump.
4. The XYZ translation panel dims — translation has no closed-loop equivalent without a position sensor.
5. The rotation sliders now set the **target orientation setpoint**, not direct servo angles.
6. The error display shows live per-axis error:
   - **`✓ LOCKED`** (green) — all axes within ±0.5° of target
   - **`▶ MOVING`** (orange) — at least one axis outside tolerance; correction in progress

### Disturbance Rejection

While in closed-loop mode, physically push or tilt the platform. The proportional controller on the ESP32 reads the resulting MPU6050 error and drives the servos back toward the setpoint. The response speed depends on the `Kp` gain — see the PID Tuning section.

### Returning to Open Loop

Click the **`🔒 CLOSED LOOP`** button. The app sends `MODE:OPEN` and the ESP32 calls `resetToZero()`, centering all servos.

---

## Communication Protocol

All packets are UTF-8 text strings terminated with `\n`. WebSocket and Serial use identical packet formats.

### PC → ESP32

| Packet | Format | Example | When sent |
|---|---|---|---|
| Mode switch | `MODE:OPEN` or `MODE:CLOSED` | `MODE:CLOSED` | Mode toggle button |
| Closed-loop target | `TARGET:roll,pitch,yaw` | `TARGET:8.5,-3.0,0.0` | Any setpoint slider change |

> In the current firmware version, open-loop servo angles are not sent as a separate packet — the `MODE:OPEN` command returns the ESP32 to home position. The IK computation result is used for 3D visualisation on the PC but is not transmitted to the ESP32 in this firmware revision.

### ESP32 → PC

| Packet | Format | Example | Rate |
|---|---|---|---|
| Sensor feedback | `FB:roll,pitch,yaw,temp,mode` | `FB:12.3,-4.1,0.0,25.0,1` | 10 Hz (every 100 ms) |
| Mode acknowledgement | `MODE_OPEN` or `MODE_CLOSED` | `MODE_CLOSED` | On mode change |

**Feedback fields:**

| Field | Unit | Source |
|---|---|---|
| roll | degrees | MPU6050 accelerometer — `atan2(ay, az)` |
| pitch | degrees | MPU6050 accelerometer — `atan2(-ax, √(ay²+az²))` |
| yaw | degrees | MPU6050 gyroscope — integrated `gyro.z × dt` |
| temp | degrees C | Fixed placeholder (25.0) in current firmware |
| mode | 0 or 1 | 0 = open loop, 1 = closed loop |

---

## Architecture Notes

### Why the PID loop runs on the ESP32

Network round-trip time over WebSocket is variable and typically 10–50 ms depending on Wi-Fi conditions. At a 30 Hz control rate, a single dropped or delayed packet consumes the entire cycle budget, introducing step disturbances into the control signal. Running the proportional controller on the ESP32 bounds the control loop latency to the sensor read time (~1 ms) with no dependence on network timing. The PC is entirely out of the correction path in closed-loop mode.

### Why the IK solver runs on the PC

The full Stewart Platform IK involves evaluating one trigonometric equation per servo per cycle — six `arcsin`, six `arctan2`, and supporting arithmetic. This is straightforward in C# with `System.Numerics` on the PC but would require careful fixed-point or floating-point management on the ESP32 alongside the sensor reads, WebSocket handling, and PWM output. More importantly, the IK result is needed for the 3D visualiser regardless, so it makes sense to keep it on the machine running the visualiser. The closed-loop correction on the ESP32 uses a simpler linear influence matrix rather than re-solving the full IK at 30 Hz.

### Why raw OpenGL replaced HelixToolkit

HelixToolkit is tightly coupled to WPF's visual tree and Windows Presentation Foundation renderer. It has no Avalonia port. When the decision was made to support Linux and Raspberry Pi — both relevant for IoT deployment in the research context — HelixToolkit was no longer an option. Silk.NET provides thin managed bindings to the native OpenGL API and works with Avalonia's `OpenGlControlBase` on all target platforms. The custom renderer uses a VAO/VBO pipeline with manually bound attribute locations to maintain compatibility with older OpenGL and OpenGL ES (ANGLE) implementations.

### Why cross-platform support was prioritised

The intended deployment context includes Raspberry Pi as the embedded control interface alongside the physical platform. A Windows-only application would require a separate PC, increasing system complexity. The Avalonia rewrite allows the control interface to run on the same embedded Linux system as the platform.

### Axis-to-Servo Influence Matrix

Rather than solving the full inverse kinematics system during every 30 Hz closed-loop cycle, orientation corrections are distributed across the six servos using a fixed linearised influence matrix implemented in `applyCorrection()`.

The matrix approximates the Stewart Platform Jacobian near the home pose and is valid within the platform's small-angle operating range.

```text
Servo 0 = +Roll                     + Yaw × 0.3
Servo 1 = -Roll                     + Yaw × 0.3

Servo 2 =             + Pitch       + Yaw × 0.3
Servo 3 =             - Pitch       + Yaw × 0.3

Servo 4 = +Roll × 0.7 + Pitch × 0.7 + Yaw × 0.3
Servo 5 = -Roll × 0.7 - Pitch × 0.7 + Yaw × 0.3
```

This is a first-order linear approximation of the actual IK Jacobian near the home position. It is accurate within the small-angle operating range used in practice and avoids the computational cost of full IK inversion on each control cycle.

---


## Known Limitations

**No forward kinematics implementation**
The system cannot compute the actual platform pose from servo angles alone. All pose information comes from the MPU6050 — which provides orientation only, not position.

**No closed-loop XYZ translation control**
The closed-loop controller operates on orientation (Roll, Pitch, Yaw) only. Translation control in X, Y, and Z requires a position sensing modality — such as a depth camera, optical flow sensor, or linear encoders — none of which are currently integrated.

**MPU6050 yaw drift**
Roll and Pitch are computed from the accelerometer and are stable over time. Yaw is computed by integrating the gyroscope Z-axis output, which accumulates drift of approximately 0.5–2°/minute depending on temperature and vibration. For long sessions, the measured yaw will diverge from the physical yaw. There is no magnetometer correction in the current implementation.

**No absolute position sensing**
The platform has no way to verify its physical position. If a servo skips steps or stalls, the IK solution diverges silently from the actual state. The only observable indicator is the MPU6050 orientation, which reflects the real platform orientation — but not which servo caused the discrepancy.


## Troubleshooting

| Symptom | Likely cause | Resolution |
|---|---|---|
| Cannot connect immediately after boot | `calibrateGyro()` blocking for 500 ms | Wait for `Ready!` in Serial Monitor before connecting |
| `MPU6050 NOT FOUND!` on boot | I2C wiring or address conflict | Check SDA/SCL connections; verify no address conflict at `0x68` |
| Platform moves to wrong position in open loop | `RobotConfig.cs` dimensions don't match physical build | Re-measure and update `BaseRadius`, `HornLength`, `RodLength`, `InitialHeight` |
| Servos twitch but platform doesn't move | `SERVO_MIDDLE_PULSE_WIDTH` values incorrect | Recalibrate each servo individually until horn is level at home |
| Platform oscillates in closed loop | `Kp` too high for this geometry | Reduce `kp` in `performClosedLoop()` by 30–50% |
| Yaw drifts in closed loop over time | Gyro integration drift (expected) | Switch to open loop and back to reset yaw accumulator; reduce Ki if used |
| 3D view blank on Windows | ANGLE/OpenGL driver issue | Update GPU drivers; confirm Avalonia can access OpenGL ES via ANGLE |
| Wi-Fi connection refused | ESP32 AP not fully initialised | Wait for `AP IP:` line in Serial Monitor before connecting |
| Serial connection drops intermittently | Insufficient USB cable or high servo current draw | Use a powered USB hub; ensure servos have their own power rail |
| Mode toggle has no visible effect on platform | Not connected | Connect to the ESP32 first — mode commands require an active connection |

---

## Important Files

| File | Version | Description |
|---|---|---|
| `stewart_avalonia_04.ino` | Both | ESP32 firmware — WebSocket, Serial, P-controller, MPU6050, PCA9685 |
| `MainWindow.axaml` | Avalonia V2 | Avalonia UI layout — all panels, sliders, error display, mode toggle |
| `MainWindow.axaml.cs` | Avalonia V2 | UI logic — connection, timers, open/closed loop, send/receive |
| `PlatformView3D.cs` | Avalonia V2 | Custom OpenGL 3D viewport — Silk.NET, VAO, VBO, cross-platform shaders |
| `StewartMath.cs` | Both | Full 6-DOF IK solver — ZYX Euler matrix, `System.Numerics` float |
| `RobotConfig.cs` | Both | Physical dimensions, attachment angles, beta angles, servo config |
| `MainWindow.xaml` | WPF V1 | WPF layout with HelixToolkit 3D viewport |
| `MainWindow.xaml.cs` | WPF V1 | WPF UI logic and HelixToolkit visual updates |

---

## Applications

The Stewart Platform architecture is used across a range of engineering and research domains. This implementation was designed with the following in mind:

- **Motion platform research** — studying parallel manipulator kinematics and closed-loop control strategies
- **IoT embedded control** — demonstrating distributed computation between an embedded microcontroller and an edge computing device
- **Robotics education** — demonstrating inverse kinematics, sensor fusion, real-time control, and cross-platform software development in a single integrated system
- **Camera stabilisation and pointing** — orientation control with disturbance rejection is directly applicable to gimbal applications

---

## Acknowledgements

- [Avalonia UI](https://avaloniaui.net/) — cross-platform .NET UI framework that made Linux and Raspberry Pi deployment possible
- [Silk.NET](https://github.com/dotnet/Silk.NET) — managed bindings for OpenGL and other native APIs, maintained by the .NET Foundation
- [HelixToolkit](https://github.com/helix-toolkit/helix-toolkit) — 3D visualisation library used in the V1 WPF implementation
- [Adafruit Industries](https://adafruit.com/) — MPU6050 and PCA9685 Arduino libraries
- [Markus Sattler — arduinoWebSockets](https://github.com/Links2004/arduinoWebSockets) — WebSocket server implementation for ESP32 / Arduino
- Sabaragamuwa University of Sri Lanka — Department of Information Systems

---

## Notice

This repository and its contents are part of an active undergraduate research project. The codebase reflects the state of the system at the time of submission and may be updated as research progresses. The firmware, desktop application, and documentation were developed and written by the project author as original research contributions.
