# DC_MotorGroup API

The `DC_MotorGroup` class simplifies controlling multiple `DC_Motor` objects as a unified group. It provides convenient methods to apply control commands, configurations, and PID constants across all motors simultaneously.

## Constructors

DC_MotorGroup supports several construction patterns for maximum flexibility:

### From Shared Pointers
```cpp
// From initializer list
explicit DC_MotorGroup(std::initializer_list<std::shared_ptr<DC_Motor>> list);

// From vector
explicit DC_MotorGroup(const std::vector<std::shared_ptr<DC_Motor>>& list);
explicit DC_MotorGroup(std::vector<std::shared_ptr<DC_Motor>>&& list);
```

### From Raw Pointers
```cpp
// From initializer list of pointers
explicit DC_MotorGroup(std::initializer_list<DC_Motor*> list);

// From vector of pointers
explicit DC_MotorGroup(const std::vector<DC_Motor*>& list);
```

### From Motor Objects
```cpp
// From initializer list of motor objects
explicit DC_MotorGroup(std::initializer_list<DC_Motor> list);

// From vector of motor objects
explicit DC_MotorGroup(const std::vector<DC_Motor>& list);
```

### From Ports (Convenience Constructor)
```cpp
// Create motors from port numbers automatically
explicit DC_MotorGroup(
    std::initializer_list<int> ports, 
    pros::v5::MotorGears gearset = util::Aliases::Blue
);

explicit DC_MotorGroup(
    const std::vector<int>& ports, 
    pros::v5::MotorGears gearset = util::Aliases::Blue
);
```

### Examples
```cpp
#include "DC_Lib/api.hpp"

// Create motors and add to group via shared pointers
auto motor1 = std::make_shared<DC_Lib::DC_Motor>(1);
auto motor2 = std::make_shared<DC_Lib::DC_Motor>(2);
DC_MotorGroup group({motor1, motor2});

// Create group directly from port numbers
DC_MotorGroup drivebase({1, 2, 3, 4});

// Create group from raw pointers
DC_Lib::DC_Motor m1(1), m2(2), m3(3);
DC_MotorGroup group({&m1, &m2, &m3});
```

## Core Control

### setTargetRPM
Set the target RPM for all motors in the group.

```cpp
void setTargetRPM(double rpm);
```

### setTargetPercent
Set the target power percentage for all motors in the group.

```cpp
void setTargetPercent(double percent);
```

### stop
Stop all motors in the group.

```cpp
void stop();
```

### resetPositions
Reset the encoder position to zero for all motors.

```cpp
void resetPositions();
```

### setBrakeMode
Set the brake mode for all motors in the group.

```cpp
void setBrakeMode(pros::v5::MotorBrake mode);
```

### Example
```cpp
DC_MotorGroup left_side({1, 2, 3});
DC_MotorGroup right_side({4, 5, 6});

// Control both groups
left_side.setTargetRPM(600);
right_side.setTargetRPM(-600);  // Opposite direction

// Stop all motors
left_side.stop();
right_side.stop();

// Set brake mode
left_side.setBrakeMode(DC_Lib::util::Aliases::Hold);
```

## PID Configuration

### setDualConstants
Set dual-speed PID constants for all motors (4 or 8 parameter versions).

```cpp
void setDualConstants(
    double kvLow, double kpLow, 
    double kvHigh, double kpHigh
);

void setDualConstants(
    double kvLow, double kpLow, double kiLow, double kdLow,
    double kvHigh, double kpHigh, double kiHigh, double kdHigh
);
```

### setLowConstants
Set low-speed PID constants for all motors.

```cpp
void setLowConstants(double kvLow, double kpLow);
void setLowConstants(double kvLow, double kpLow, double kiLow, double kdLow);
```

### setHighConstants
Set high-speed PID constants for all motors.

```cpp
void setHighConstants(double kvHigh, double kpHigh);
void setHighConstants(double kvHigh, double kpHigh, double kiHigh, double kdHigh);
```

### setStartI
Initialize the integral error term for all motors.

```cpp
void setStartI(double startIntegral);
```

### setPIDEnabled
Enable or disable PID control for all motors.

```cpp
void setPIDEnabled(bool enabled);
bool isAnyPIDEnabled() const;      // True if any motor has PID enabled
bool areAllPIDEnabled() const;     // True if all motors have PID enabled
```

### Example
```cpp
DC_MotorGroup drivetrain({1, 2, 3, 4, 5, 6});

// Configure PID for entire drivetrain
drivetrain.setDualConstants(
    0.05, 0.1,      // Low speed
    0.05, 0.05      // High speed
);

// Disable PID for a specific test
drivetrain.setPIDEnabled(false);

// Check PID status
if (drivetrain.areAllPIDEnabled()) {
    printf("All motors have PID enabled\n");
}
```

## Slew Control

### setSlewLimitEnabled
Enable slew rate limiting for all motors.

```cpp
void setSlewLimitEnabled(bool SlewOn);
```

### setSlewRate
Set the slew rate for all motors.

```cpp
void setSlewRate(double Slew);
```

### Example
```cpp
drivetrain.setSlewLimitEnabled(true);
drivetrain.setSlewRate(300.0);  // Limit speed change to 300 RPM per iteration
```

## Load and Torque Control

### setLoadCompensation (3 overloads)
Enable load compensation for all motors in the group.

```cpp
void setLoadCompensation(bool enabled);
void setLoadCompensation(bool enabled, double kBoost);
void setLoadCompensation(bool enabled, double kBoost, int threshold);
```

### setMinTorque (3 overloads)
Set minimum torque for all motors.

```cpp
void setMinTorque(bool enabled);
void setMinTorque(bool enabled, int minMv);
void setMinTorque(bool enabled, int minMv, int minLowMv);
```

### Example
```cpp
drivetrain.setLoadCompensation(true, 5.0, 15);
drivetrain.setMinTorque(true, 1200, 600);
```

## Status and Telemetry

### getAverageRPM
Get the average RPM across all motors in the group.

```cpp
double getAverageRPM() const;
```

### isAnySpinning
Check if any motor in the group is currently spinning at target.

```cpp
bool isAnySpinning() const;
```

### getMotors
Get access to the underlying motor collection.

```cpp
const std::vector<std::shared_ptr<DC_Motor>>& getMotors() const;
```

### size

```cpp
size_t size() const;
```

### Example
```cpp
printf("Average RPM: %f\n", drivetrain.getAverageRPM());

if (drivetrain.isAnySpinning()) {
    printf("One or more motors are spinning\n");
}

printf("Group has %zu motors\n", drivetrain.size());

// Access individual motors
for (const auto& motor : drivetrain.getMotors()) {
    printf("Motor temperature: %f C\n", motor->getTemperature());
}
```

## Complete Example: Arcade Drive

```cpp
#include "main.h"
#include "DC_Lib/api.hpp"

void opcontrol() {
    // Create left and right motor groups
    DC_MotorGroup left_side({1, 2, 3});
    DC_MotorGroup right_side({4, 5, 6});
    
    // Configure motor groups
    left_side.setDualConstants(0.05, 0.1, 0.05, 0.05);
    right_side.setDualConstants(0.05, 0.1, 0.05, 0.05);
    
    left_side.setBrakeMode(DC_Lib::util::Aliases::Hold);
    right_side.setBrakeMode(DC_Lib::util::Aliases::Hold);
    
    left_side.setSlewLimitEnabled(true);
    right_side.setSlewLimitEnabled(true);
    left_side.setSlewRate(400.0);
    right_side.setSlewRate(400.0);
    
    left_side.setLoadCompensation(true, 5.0);
    right_side.setLoadCompensation(true, 5.0);
    
    pros::Controller master(pros::E_CONTROLLER_MASTER);
    
    while (true) {
        // Arcade control scheme
        int forward = master.get_analog(ANALOG_LEFT_Y);   // Forward/backward
        int turn = master.get_analog(ANALOG_RIGHT_X);     // Turn
        
        // Calculate power for each side
        int left_power = forward + turn;
        int right_power = forward - turn;
        
        // Apply power
        left_side.setTargetPercent(left_power);
        right_side.setTargetPercent(right_power);
        
        // Telemetry
        printf("L: %f, R: %f\n", left_side.getAverageRPM(), right_side.getAverageRPM());
        
        pros::delay(20);
    }
}
```

## Complete Example: Subsystem Tank Drive

```cpp
#include "main.h"
#include "DC_Lib/api.hpp"

class Drivetrain {
private:
    DC_MotorGroup left_motors;
    DC_MotorGroup right_motors;
    
public:
    Drivetrain(std::initializer_list<int> left_ports, 
               std::initializer_list<int> right_ports)
        : left_motors(left_ports), right_motors(right_ports) {
        
        // Configure both sides
        left_motors.setDualConstants(0.05, 0.1, 0.05, 0.05);
        right_motors.setDualConstants(0.05, 0.1, 0.05, 0.05);
        
        left_motors.setBrakeMode(DC_Lib::util::Aliases::Hold);
        right_motors.setBrakeMode(DC_Lib::util::Aliases::Hold);
    }
    
    void arcadeDrive(double forward, double turn) {
        left_motors.setTargetPercent(forward + turn);
        right_motors.setTargetPercent(forward - turn);
    }
    
    void stop() {
        left_motors.stop();
        right_motors.stop();
    }
};

void opcontrol() {
    Drivetrain drive({1, 2, 3}, {4, 5, 6});
    pros::Controller master(pros::E_CONTROLLER_MASTER);
    
    while (true) {
        int forward = master.get_analog(ANALOG_LEFT_Y);
        int turn = master.get_analog(ANALOG_RIGHT_X);
        
        drive.arcadeDrive(forward, turn);
        
        pros::delay(20);
    }
}