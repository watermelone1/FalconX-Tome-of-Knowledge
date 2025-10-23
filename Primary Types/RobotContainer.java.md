The `RobotContainer` class *contains* much of the information about the robot. This is where you define Subsystems, set up controller bindings, define autonomous routines and prepare commands. Most properties about the robot will be located here.

Much of `RobotContainer` should be self-explanatory but I will mention some notable things. This example is a simplified version of the 2025 `RobotContainer.java`.

```java
public class RobotContainer {
	private final Drivetrain drivetrain = new Drivetrain(...);
	private final Intake intake = new Intake(...);
	// ...
	UsbCamera intakeCam;
	Settings settings;
	
	private final CommandXboxController driverController = new CommandXboxController(...);
	private final CommandXboxController operatorController = new CommandXboxController(...);
	//...
	
	Command swerveDrive = new SwerveDriveCommand(drivetrain, driverController);
	Command speedModeDrive = //... ok this was a weird but effective way to drive the robot don't question it
	Command intake = new IntakeCommand(intake, operatorController);
	//...
	
	private SendableChooser<Command> autoChooser = new SendableChooser<>();
	
	public RobotContainer() {
		Util.setStartTime(LocalDateTime.now()); // Sets the starting time of the robot
		DataLogManager.start(Filesystem.getOperatingDirectory() + "/logs", Util.getLogFilename());
		
		drivetrain.setupPathPlanner();
	    NamedCommands.registerCommand("Go To L3", new GoToArmPosition(Position.L3, arm, elevator));
	    NamedCommands.registerCommand("Go to L2", new GoToArmPosition(Position.L2, arm, elevator));
	    //...
	    
	    autoChooser = AutoBuilder.buildAutoChooser();
	    SmartDashboard.putData("Auto Chooser", autoChooser);
	    
	    configureBindings();
	    
	    intakeCam = CameraServer.startAutomaticCapture();
	    intakeCam.setFPS(20);
	}
	
	private void configureBindings() {
		settings.driverSettings.speedModeButton.whileTrue(speedModeDrive);
		drivetrain.setDefaultCommand(swerveDrive);
		//...
		
		settings.operatorSettings.intakeButton.whileTrue(new GrabCoral(intake, settings, operatorController));
		//...
		arm.setDefaultCommand(new MoveArm(arm, operatorController));
		elevator.setDefaultCommand(new ManualElevator(elevator, operatorController));
	}
	
	public Command getAutonomousCommand() {
		return autoChooser.getSelected();
	}
}
```

There's a lot going on here but nothing complicated. We start, like always, by defining variables. Here we have a `SendableChooser`, which allows the user to easily switch between different options using SmartDashboard, in this case we use one to choose between auto routines.

```java
private final Drivetrain drivetrain = new Drivetrain(...);
private final Intake intake = new Intake(...);
// ...
UsbCamera intakeCam;
Settings settings;

private final CommandXboxController driverController = new CommandXboxController(...);
private final CommandXboxController operatorController = new CommandXboxController(...);
//...

Command swerveDrive = new SwerveDriveCommand(drivetrain, driverController);
Command speedModeDrive = //... ok this was a weird but effective way to drive the robot don't question it
Command intake = new IntakeCommand(intake, operatorController);
//...

private SendableChooser<Command> autoChooser = new SendableChooser<>();
```

Next, like always, we have the constructor. We use it to set up different components such as auto with `NamedCommands` and `setupPathPlanner()`. We also set up logging with `DataLogMananger.start()`

```java
public RobotContainer() {
	Util.setStartTime(LocalDateTime.now()); // Sets the starting time of the robot
	DataLogManager.start(Filesystem.getOperatingDirectory() + "/logs", Util.getLogFilename());
	
	drivetrain.setupPathPlanner();
	NamedCommands.registerCommand("Go To L3", new GoToArmPosition(Position.L3, arm, elevator));
	NamedCommands.registerCommand("Go to L2", new GoToArmPosition(Position.L2, arm, elevator));
	//...
	
	autoChooser = AutoBuilder.buildAutoChooser();
	SmartDashboard.putData("Auto Chooser", autoChooser);
	
	configureBindings();
	
	intakeCam = CameraServer.startAutomaticCapture();
	intakeCam.setFPS(20);
}
```

Next we set up controllers in `configureBindings()`. Much of the code for specific bindings is in the `Settings` class which we used that year.

```java
private void configureBindings() {
	settings.driverSettings.speedModeButton.whileTrue(speedModeDrive);
	drivetrain.setDefaultCommand(swerveDrive);
	//...
	
	settings.operatorSettings.intakeButton.whileTrue(new GrabCoral(intake, settings, operatorController));
	//...
	arm.setDefaultCommand(new MoveArm(arm, operatorController));
	elevator.setDefaultCommand(new ManualElevator(elevator, operatorController));
}
```

Last we choose the autonomous routine with `getAutonomousCommand()`. This method is called just before the start of a match and the command is run.

```java
public Command getAutonomousCommand() {
	return autoChooser.getSelected();
}
```