While a subsystem directly controls the motors and sensors. A Command tells subsystems when to run the motors and responds to sensors. They control the logic to control the robot. Every subsystem should be running a command at all times, even if that command only says to do nothing. A command can run multiple subsystems at a time.
## Command Class

A Command class is made by extending `Command`

Commands have some important methods that should be overridden. Notably `execute()`, `initialize()`, `isFinished()`, and `end()`

I will show po0with an example command and then explain its components. This example is a simplified version of the `AutoRotate` command from our 2024 robot. This command would orient the robot to the correct target by using the camera to find the angle to the target, and then automatically rotating to it

```java
public class AutoRotate extends Command {
    Drivetrain drivetrain;
    Vision vision;
    PigeonIMU gyro;
    double rotateAngle;
    double distanceToRotate;
    PIDController pid = new PIDController(0.03, 0, 0.005);
    double feedforward = 0.2;
    
    public AutoRotate (Drivetrain drivetrain, Vision vision) {
        drivetrain = drivetrain;
        vision = vision;
        gyro = drivetrain.getGyro();
        addRequirements(drivetrain); // Don't put vision in requirements or else the command will be canceled
    }
    
    @Override
    public void initialize() {
        if (vision.hasCorrectTargets()) {
            distanceToRotate = -vision.getYaw().get(); // Measured in degrees
        }
        double cameraOffset = 10; // Not sure what this number meant, i think it was the angle the camera was pointed at
        
        pid.setSetpoint(gyro.getYaw() + distanceToRotate + cameraOffset); // Set up PID
        pid.setTolerance(0.5);
    }

    @Override
    public void execute () {
        if (vision.hasCorrectTargets()) {
            distanceToRotate = -vision.getYaw().get();
        }
	    
	    // Calculate rotation speed
        double rotateSpeed = pid.calculate(gyro.getYaw());
        rotateSpeed += feedforward * Math.signum(rotateSpeed);
        rotateSpeed = MathUtil.clamp(rotateSpeed, -1, 1);
        
        // If its already there dont move
        if (pid.atSetpoint()) {rotateSpeed = 0;}
        
        // move
        drivetrain.curvatureDrive(0, rotateSpeed, false, true);
        
        // Log results
        DashboardHelper.putNumber(LogLevel.Debug, "Auto Rotate Rotate Speed", rotateSpeed);
        DashboardHelper.putNumber(LogLevel.Debug, "Auto Rotate Distance from setpoint", pid.getPositionError());
    }
    
    @Override
    public boolean isFinished () {
        if (!mision.hasCorrectTargets()) return true; // Camera has no targets, cannot orient properly
        return Math.abs(distanceToRotate) < 0.02; // Robot is close enough
    }
}
```

Start at the top. We start by defining the variables that are necessary. This used the old gyro called the Pigeon. We had subsystems called Drivetrain and Vision. It also used PID to control the angle.

```java
public class AutoRotate extends Command {
    Drivetrain drivetrain;
    Vision vision;
    PigeonIMU gyro;
    double rotateAngle;
    double distanceToRotate;
    PIDController pid = new PIDController(0.03, 0, 0.005);
    double feedforward = 0.2;
```

Next the constructor, when we create the command, we supply it with the subsystems it needs. we also use `addRequirements()` to tell the drivetrain that this command is being run. It is important to note that when you call a command it **automatically cancels** any other command that subsystem is using. In this case, the vision has its own command that needs to run at all times, for this reason we don't add it to requirements, usually this is not best practice and should be avoided.

```java
    public AutoRotate(Drivetrain drivetrain, Vision vision) {
        drivetrain = drivetrain;
        vision = vision;
        gyro = drivetrain.getGyro();
        addRequirements(drivetrain); // Don't put vision in requirements or else the command will be canceled
    }
```

The `initialize()` method is automatically called the *first* time to command is scheduled. We use it to set up any variables before the command runs. In this example we use it to set up the correct angle as well as set up PID.

```java
    @Override
    public void initialize() {
        if (vision.hasCorrectTargets()) {
            distanceToRotate = -vision.getYaw().get(); // Measured in degrees
        }
        double cameraOffset = 10; // Not sure what this number meant, i think it was the angle the camera was pointed at
        
        pid.setSetpoint(gyro.getYaw() + distanceToRotate + cameraOffset); // Set up PID
        pid.setTolerance(0.5);
    }
```

The `execute()` method runs every time the command is scheduled, every 20ms (50/s). It is meant to handle the logic of running the command. Most things that control the subsystems should be part of the `execute()` method.

```java
    @Override
    public void execute() {
        if (vision.hasCorrectTargets()) {
            distanceToRotate = -vision.getYaw().get();
        }
	    
	    // Calculate rotation speed
        double rotateSpeed = pid.calculate(gyro.getYaw());
        rotateSpeed += feedforward * Math.signum(rotateSpeed);
        rotateSpeed = MathUtil.clamp(rotateSpeed, -1, 1);
        
        // If its already there dont move
        if (pid.atSetpoint()) {rotateSpeed = 0;}
        
        // move
        drivetrain.curvatureDrive(0, rotateSpeed, false, true);
        
        // Log results
        DashboardHelper.putNumber(LogLevel.Debug, "Auto Rotate Rotate Speed", rotateSpeed);
        DashboardHelper.putNumber(LogLevel.Debug, "Auto Rotate Distance from setpoint", pid.getPositionError());
    }
```

`isFinished()` does not control normal robot logic. It tells the command when to stop. It is called before `execute()` and if it returns `true` then the command ends and calls the `end()` function. If any thing needs to happen when the command ends, make sure to include `end()`. In this case, the command should end automatically if there are no targets, or if the robot is close enough to make the shot.

```java
    @Override
    public boolean isFinished() {
        if (!mision.hasCorrectTargets()) return true; // Camera has no targets, cannot orient properly
        return Math.abs(distanceToRotate) < 0.02; // Robot is close enough
    }
```

There are a few different ways to use commands. All subsystems should have a default command using `Subsystem.setDefaultCommand()`. This will run the command as long as no others override it. It also will restart once it is able to.

You can run a command when a button is pressed. The controllers have a `Trigger` for each button. Use the `Trigger.onTrue()` or `Trigger.whileTrue()` or others to start commands when a button is pressed.

## Command Compositions

Commands have special methods allowing you to combine multiple at once.

`foo.repeatedly()` runs `foo`, restarting the command when it finishes, until it is interrupted.
`foo.andThen(bar)` runs `foo` first and `bar` after foo is finished
`foo.alongWith(bar)` runs `foo` and `bar` at the same time
`foo.withTimeout(3)` runs foo for 3 seconds
`foo.until(bar)` runs until bar returns true where bar is a [lambda expression](https://docs.oracle.com/javase/tutorial/java/javaOO/lambdaexpressions.html#syntax)

There are also special types of commands that run under certain conditions

`Commands.parallel(cmdA, cmdB, cmdC)` runs all three at the same time. The overall command runs until all three have finished. Equivalent to `cmdA.alongWith(cmdB).alongWith(cmdC)`
`Commands.race(cmdA, cmdB, cmdC)` runs all three at the same time, but ends all of them once any of them end. Equivalent to `cmdA.raceWith(cmdB, cmdC)`
`Commands.deadline(cmdA, cmdB, cmdC)` runs all three at the same time, but ends all of them once the *first* one (the deadline) ends. Equivalent to `cmdA.deadlineFor(cmdB, cmdC)`

For example, we used these extensively in 2024 especially in the auto routine

```java
NamedCommands.registerCommand("Intake", 
      new RunIntake(m_intake, -0.6) // Run the intake at -0.6 speed
        .alongWith(new ArmGoToGoalRotation(m_arm, 0).withTimeout(.1)) // keep the arm at the ground
        .alongWith(new RunIndex(m_index, 0.75)) // run the index at 0.75 speed
        .until(() -> {return m_sensor.getNoteSensed();}) // do all of that until we pick up a note
        .andThen( // after that...
          new RunIndex(m_index, -.3) // run the index at -0.3 speed
          .withTimeout(.15) // for a fraction of a second to realign the note
        ).alongWith( // at the same time...
          new SimpleShoot(m_shooter, -.2) // run the shooter backwards
          .withTimeout(.15) //for a fraction of a second, so the note doesnt get stuck
        )
      );
```
