A PID Controller (Proportional - Integral - Derivative) is a way to automatically move and manage a value such as position, or rotation, or distance, and adjust to keep it in place by using feedback.

## Theory
---

For Example:

Lets say you are the robot and you need to get from Point A to Point B as fast as possible. One method is to run the motors at max speed until, you reach the destination, then turn off the motors.

```java
void run() {
	if (distanceToDestination > 0) {
		motorSpeed = 100;
	} else {
		motorSpeed = 0;
	}
}
```

This will quickly get you where you want, but quickly overshoot by a lot. 

Another method would be to run motors until you reach the destination and backwards when you overshoot. 

```java
void run() {
	if (distanceToDestination > 0) {
		motorSpeed = 100;
	} else {
		motorSpeed = -100;
	}
}
```

This will keep you near the point you want to be at but again, again, will overshoot the other way

Now we need to slow down when we get near the point, if we use the distance *as* the speed, then it will be slower near the destination and higher further away. 

```java
void run() {
	double k = 0.3; // Some constant
	motorSpeed = distanceToDestination * k;
}
```

Things are looking promising now, but we seem to stop short, this could be due to a number of reasons, friction, voltage limits on the motors, etc. Additionally, it can be very hard to tune the acceleration using this method

There is a powerful way to control it using a PID Controller.

A PID Controller uses 3 parts, the P, the I, and the D. All three parts add together to decide the speed. Each part has a constant usually called $K_p$ $K_i$ and $K_d$ . The P part is the same as the 3rd example, with k equal to $K_p$ , it takes the distance and multiplies it by a constant. The D part (the derivative) basically says "slow down if you will overshoot". If you are about to go past the target, the D part will be lower, slowing you down. Lastly the I part (the integral) basically says "If you haven't been at the target, speed up". If you take a long time away from the target, the I part will be higher.

## Tuning

These together allow you to react to different scenarios in, all in one controller. The next problem comes with finding the constants for P I and D. There is usually no easy way to calculate these and the optimal numbers may be extremely small or extremely large. In general you start by finding P.

Set $K_i$ and $K_d$ to 0 and choose an arbitrary constant for $K_p$ . If you overshoot your target then lower P, if you undershoot then raise P. Continue until it reaches the target.

Next tune $K_d$ . When D is too high, you will oscillate around the target, too low and you won't get to the target quick enough. D also helps react to a moving target.

$K_i$ is a weird one. In a lot of cases it is fine to leave it at 0, however you may want to adjust it if you have a long distance or it takes a while to reach the target. A high value of I can reach the target faster, but overshoot more, a low value will can slow down to the target faster but oscillate more.

(Tip: When tuning $K_i$ , try starting with roughly 10% of $K_p$)

![[PID Tuning.gif|500]]

## Programming
---

I will write two examples using the PID Controller. The first will hold a rotating arm at an angle, and the second will drive a tank drive to a certain distance.

### Example 1: Rotating Arm

Start with a basic arm class.

```java
public class Arm extends SubsytemBase {
	ExampleMotor motor = new Motor();
	double targetAngle = 90.0; // 90 degrees
	
	public void setMotorSpeed(double speed) {
		motor.set(speed);
	}
	
	public double getAngle() {
		return motor.getAngle();
	}
	
	public void setTargetAngle(double targetAngle) {
		this.targetAngle = targetAngle;
	}
}
```

Now lets create a `PIDController`. We supply it with the target called the setpoint, and the constants `kP`, `kI`, and `kD`.

```java
//...
double targetAngle = 90.0;
PIDController pid = new PIDController();
final double kP = 0.0;
final double kI = 0.0;
final double kD = 0.0;

public Arm() {
	// initialize pid
	pid.setSetpoint(targetAngle);
	pid.setP(kP);
	pid.setI(kI);
	pid.setD(kD);
}
//...
```

Now we update our PID Controller every loop and tell it our new position

```java
//...
@Override
public void periodic() {
	double calculatedSpeed = pid.calculate(getAngle()); // This updates the pid controller and gets the last calculation.
	setMotorSpeed(calculatedSpeed);
}
```

![[The PID Loop.png|500]]

Finally we tune PID values

```java
//...
final double kP = 0.05;
final double kI = 0.0001;
final double kD = 0.003;
```

### Example 2: Drive a Distance

Once again, start with a Tank Drive

```java
public class TankDrivetrain extends SubsystemBase {
	ExampleMotor leftMotors = new ExampleMotor();
	ExampleMotor rightMotors = new ExampleMotor();
	
	public void setSpeed(double speed) {
		leftMotors.set(speed);
		rightMotors.set(speed);
	}
	
	public double getDistance() {
		// Implementation not shown
	}
}
```

Driving a distance is an action the robot does, so it should be a command

```java
public class DriveDistance extends Command {
	TankDrivetrain tankDrive;
	double distance;
	
	public DriveDistance(TankDrivetrain tankDrive, double distance) {
		this.tankDrive = tankDrive;
		this.distance = distance;
		
		addRequirements(tankDrive);
	}
	
	public void execute() {}
	
	public void end(boolean interrupted) {}
	
	public boolean isFinished() {return false;}
}
```

At the moment the command does nothing, lets start having it drive the robot forward until it reaches the correct distance

```java
public void execute() {
	tankDrive.setSpeed(1.0); // Drive forward
}

public void end(boolean interrupted) {
	tankDrive.setSpeed(0.0); // Stop at the target
}

public boolean isFinished() {
	if (Math.abs(tankDrive.getDistance() - distance)  <= 0.01) { // Distance is close enough
		return true;
	} else {
		return false;
	}
}
```

This will stop at the target but runs into the same problem as stated in the beginning, it overshoots the target. We can solve this by implementing a PID Controller

```java
TankDrivetrain tankDrive;
double distance;

PIDController pid = new PIDController;
final double kP = 0.0;
final double kI = 0.0;
final double kD = 0.0;

public DriveDistance(TankDrivetrain tankDrive, double distance) {
	//...
	pid.setSetpoint(distance); // Tell the PID Controller what the target is
	pid.setP(kP);
	pid.setI(kI);
	pid.setD(kD);
}

public void execute() {
	double calculatedSpeed = pid.calculate(tankDrive.getDistance()); // Update PID loop and get value
	tankDrive.setSpeed(calculatedSpeed);
}
```

Now tune PID

```java
final double kP = 0.4;
final double kI = 0.005;
final double kD = 0.06;
```

At this point, ideally the robot would drive forward to the targeted distance. But there's more that you can do with a PID Controller, for example, the I term can get very high in some cases, which can be helped with `PIDController.setIZone()` which disables the I term until it reaches a certain distance from the target or `setTolerance()` and `atSetpoint()` which can tell you when you are close. Implementing these are not strictly necessary but can optimize the speed to the target.

```java
public DriveDistance(TankDrivetrain tankDrive, double distance) {
	//...
	pid.setIZone(5.0);
	pid.setTolerance(0.01);
}

//...

public boolean isFinished() {
	return pid.atSetpoint();
}
```
## Technical
---

The PID Controller calculates an "error function" $e(t)$ which is the difference between the current position and the target called the "setpoint". At every step it produces three values and adds them together to be the optimal "output" that should be applied to the system. Then the loop is ran again, three values are added together to get the next output forever, settling on the setpoint.

The first value $P$  is calculated by multiplying a constant $K_p$ by the error $e(t)$ . 

The second value $I$ is calculated by multiplying a constant $K_i$ by the *integral* of the error $\int_{0}^{t}e(x)dx$  

The third value $D$ is calculated by multiplying a constant $K_d$ by the *derivative* of the error $\frac{de(t)}{dx}$ 

All the values are summed together for the final output.