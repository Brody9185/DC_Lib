# FAQ & Troubleshooting — DC_Lib

This page collects common questions and solutions when using `DC_Lib`, `DC_Motor`, and `DC_MotorGroup`.

## Q: Which header should I include?
A: Use the unified API header:

```cpp
#include "DC_Lib/api.hpp"
```

## Q: My motors spin in the wrong direction.
A: Check motor polarity and wiring. If needed, invert the motor in software:

```cpp
motor.setReversed(true);
```

For `DC_MotorGroup`, invert an entire group by configuring each motor or applying a sign to targets.

## Q: Encoder positions are incorrect or not zeroed.
A: Call `resetPositions()` on the `DC_Motor` or `DC_MotorGroup` after initializing sensors:

```cpp
motor.resetPositions();
// or
group.resetPositions();
```

Ensure your encoders are connected and configured for the correct port/encoder type.

## Q: PID controller behaves poorly (oscillation or slow response).
A: Common fixes:
- Verify the motor gearing and wheel size settings match physical hardware.
- Start with conservative PID constants; use `setLowConstants()` first.
- Try reducing `ki` to reduce integral windup.
- Use `setStartI()` to seed the integral term if needed.

Example:

```cpp
drivetrain.setLowConstants(0.04, 0.08);
drivetrain.setHighConstants(0.06, 0.05);
```

## Q: Motors overheat quickly.
A: Causes and actions:
- High continuous current: reduce max voltage/power or use gearing appropriate for load.
- Heavy stall torque: increase `setMinTorque()` threshold carefully.
- Verify `setLoadCompensation()` settings; excessive boost can increase heat.

## Q: Telemetry values seem wrong (RPM, temperature).
A: Verify sensors are reporting expected values. Use the `getAverageRPM()` and individual motor getters for debugging. Add logging at startup and before/after control changes.

## Q: My motor group commands apply unevenly across motors.
A: Check:
- That all motors use the same gearset and encoder configuration.
- Individual motor calibration (replace or reposition if faulty).
- For `DC_MotorGroup`, ensure all motors are added in the correct order and none are physically slipping.

## Q: setTargetPercent vs setTargetRPM — which to use?
A: Use `setTargetPercent()` for open-loop control or simple teleop. Use `setTargetRPM()` when closed-loop speed regulation is required (PID enabled).

## Q: How to enable/disable slew rate?
A: Use `setSlewLimitEnabled(true/false)` and tune `setSlewRate()` to prevent sudden power changes.

## Q: Build or linker errors referencing the old library names.
A: Clean your build artifacts and ensure you updated includes to `DC_Lib/api.hpp`. Also update any CMake/Makefiles or project settings that referenced old target/library names.

## Q: Where can I find example code?
A: See the `docs/t-lib/` folder for examples using `DC_Motor` and `DC_MotorGroup`.

## Still stuck?
Open an issue with the following information:
- Hardware used (motor model, gearset, encoder type).
- Short code snippet demonstrating the problem.
- Telemetry output (RPM, temperature) and observed behavior.


---

Last updated: April 23, 2026
