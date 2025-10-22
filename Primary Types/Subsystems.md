A Subsystem is a class that represents a physical part on a robot such as the Intake or Drivetrain. They handle logic for controlling motors, sensors, pneumatics, etc. Usually one of each subsystem is created in Robot Container.

A subsystem class is created by extending `SubsystemBase`

Example Tank Drive class:
```java
public class TankDrivetrain extends SubsystemBase {
	private ExampleMotor leftMotors = new ExampleMotor();
	private ExampleMotor rightMotors = new ExampleMotor();
	
	private void setLeftMotors(double velocity) {
		leftMotors.set(velocity);
	}
	
	private void setRightMotors(double velocity) {
		rightMotors.set(velocity);
	}
	
	public void setMotors(double leftVelocity, double rightVelocity) {
		leftMotors.set(leftVelocity);
		rightMotors.set(rightMotors);
	}
	
	public void arcadeDrive(double speed, double rotation) {
		speed = MathUtil.clamp(speed, -1.0, 1.0);
		rotation = MathUtil.clamp(rotation, -1.0, 1.0);
		
		double leftSpeed = speed - rotation;
		double rightSpeed = speed + rotation;
		
		setMotors(leftSpeed, rightSpeed);
	}
	
	@Override
	public void periodic() {
		updatePosition();
		
		Logger.log("Left Position",   leftMotors.getPosition());
		Logger.log("Right Position", rightMotors.getPosition());
	}
	
	private void updatePosition() {
		// ...
	}
}

public class RobotContainer {
// ...
TankDrive drivetrain = new TankDrivetrain();
Command arcadeDrive = new ArcadeDriveCommand(drivetrain, driverXbox);
drivetrain.setDefaultCommand(arcadeDrive));
// ...
}
```

Let's walk through this

This defines the class and the sets of motors on each side of the drivetrain. In reality `ExampleMotor` would be whichever type of motor controller you use on the robot such as `SparkMax` or `TalonFX`
```java
public class TankDrivetrain extends SubsystemBase {
	private ExampleMotor leftMotors = new ExampleMotor();
	private ExampleMotor rightMotors = new ExampleMotor();
```

These define methods to control individual sides of the robot. In general we control both sides at once so these are made private.

```java
private void setLeftMotors(double velocity) {
	leftMotors.set(velocity);
}

private void setRightMotors(double velocity) {
	rightMotors.set(velocity);
}
```

Usually we want to control both sides of the robot at once so we can write a method such as this one to accomplish that.

```java
public void setMotors(double leftVelocity, double rightVelocity) {
	leftMotors.set(leftVelocity);
	rightMotors.set(rightMotors);
}
```

It might be helpful to us to have multiple ways to control the robot. Arcade Drive is a system where one joystick controls the linear speed (forward and backward), while the other controls the angular speed (turning). This example comes from the [WPILib Differential Drive Class](https://github.com/wpilibsuite/allwpilib/blob/17cae787e7100b4e85dd54e70982d990c141b75c/wpilibj/src/main/java/edu/wpi/first/wpilibj/drive/DifferentialDrive.java#L261).

```java
public void arcadeDrive(double speed, double rotation) {
	speed = MathUtil.clamp(speed, -1.0, 1.0);
	rotation = MathUtil.clamp(rotation, -1.0, 1.0);
	
	double leftSpeed = speed - rotation;
	double rightSpeed = speed + rotation;
	
	setMotors(leftSpeed, rightSpeed);
}
```

Subsystems have a useful method that you can override: `periodic()`. It is automatically called every 20ms (50/s). You can use it for anything that the component should always be doing such as updating the state of its parts or logging information.

```java
@Override
public void periodic() {
	updatePosition();
	
	Logger.log("Left Position",   leftMotors.getPosition());
	Logger.log("Right Position", rightMotors.getPosition());
}

private void updatePosition() {
	// ...
}
```

When we create a subsystem you should always provide a default command which it runs when it is not scheduled to run another command. In this case we have an example Arcade Drive command (Code not shown) which we want to use when driving normally.

```java
public class RobotContainer {
// ...
	TankDrive drivetrain = new TankDrivetrain();
	Command arcadeDrive = new ArcadeDriveCommand(drivetrain, driverXbox);
	drivetrain.setDefaultCommand(arcadeDrive));
// ...
}
```

