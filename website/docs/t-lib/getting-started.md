# Getting Started with DC_Lib

DC_Lib is a custom C++ robotics library designed for VEX Robotics platforms. It provides advanced motor control capabilities with features like PID control, slew limiting, and load compensation, built on top of the PROS kernel.

## What is DC_Lib?

DC_Lib abstracts away complex motor control logic and provides an elegant, high-level API for:

- **Individual Motor Control** (`DC_Motor`): Advanced PID control with dual speed constants, load compensation, and anti-stall features
- **Motor Group Management** (`DC_MotorGroup`): Unified control of multiple motors with synchronization
- **Utilities**: Unit conversion functions, wheel size constants, and convenient aliases

## Installation

1. Include the DC_Lib header in your project:
```cpp
#include "DC_Lib/api.hpp"
```

2. Link against the DC_Lib library (typically handled by your build system)

## Basic Usage Example

Here's a simple example to get started:

```cpp
#include "main.h"
#include "DC_Lib/api.hpp"

void opcontrol() {
    // Create a single motor on port 1
    DC_Lib::DC_Motor motor(1);
    
    // Set the motor to 50% power
    motor.setTargetPercent(50_perc);
    
    // Read the motor's RPM
    double rpm = motor.getRPM();
    
    // Stop the motor
    motor.stop();
}
```

## Key Components

### DC_Motor
The `DC_Motor` class provides advanced control over a single V5 motor with:
- Real-time PID-based speed control
- Dual-speed PID constants (low and high speed regimes)
- Slew rate limiting
- Load compensation
- Minimum torque (anti-stall) features
- Background control task for non-blocking operation

### DC_MotorGroup
The `DC_MotorGroup` class manages multiple `DC_Motor` objects:
- Apply control commands to all motors at once
- Set PID constants, brake modes, and compensation settings across the group
- Query average RPM and status across the group

### Utilities
DC_Lib utilities provide:
- Accurate wheel size constants for common VEX wheels
- Convenient aliases for PROS enumerations
- Unit conversion helpers (inches ↔ mm, degrees ↔ radians)
- Type-safe percentage values with custom literal operators (`50_perc`)

## Control Modes

DC_Lib motors support two primary control modes:

- **RPM Control**: `setTargetRPM(rpm)` - Set a target rotation speed in RPM
- **Percent Power**: `setTargetPercent(percent)` - Set power from 0-100%

Both modes use background PID control for smooth, stable operation.

## Next Steps

- Learn about [DC_Motor API](./dc-motor.md)
- Explore [DC_MotorGroup API](./dc-motor-group.md)
- Check out [Utilities](./utilities.md)
- See [Common Patterns](./common-patterns.md) for real-world examples
