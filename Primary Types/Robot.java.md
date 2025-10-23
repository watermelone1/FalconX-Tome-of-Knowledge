Surprisingly, the `Robot` class is rarely important. It runs a lot of behind the scenes actions, such as creating the RobotContainer and running the Command Scheduler. 



```java
public class Robot extends TimedRobot
{

  private static Robot   instance;
  private        Command m_autonomousCommand;

  private RobotContainer robotContainer;

  private Timer disabledTimer;

  public Robot()
  {
    instance = this;
  }

  public static Robot getInstance()
  {
    return instance;
  }

  @Override
  public void robotInit()
  {
    robotContainer = new RobotContainer();
    disabledTimer = new Timer();
  }

  @Override
  public void robotPeriodic()
  {
    robotContainer.robotPeriodic();
    CommandScheduler.getInstance().run();
  }

  @Override
  public void disabledInit()
  {
    robotContainer.setMotorBrake(true);
    disabledTimer.reset();
    disabledTimer.start();
  }

  @Override
  public void disabledPeriodic()
  {
    if (disabledTimer.hasElapsed(Constants.DrivebaseConstants.WHEEL_LOCK_TIME))
    {
      robotContainer.setMotorBrake(false);
      disabledTimer.stop();
    }
  }

  @Override
  public void autonomousInit()
  {
    robotContainer.setMotorBrake(true);
    m_autonomousCommand = robotContainer.getAutonomousCommand();

    if (m_autonomousCommand != null)
    {
      m_autonomousCommand.schedule();
    }
  }

  @Override
  public void autonomousPeriodic()
  {
  }

  @Override
  public void teleopInit()
  {
    if (m_autonomousCommand != null)
    {
      m_autonomousCommand.cancel();
    } else
    {
      CommandScheduler.getInstance().cancelAll();
    }
    robotContainer.setDriveMode();
    robotContainer.setMotorBrake(true);
  }

  @Override
  public void teleopPeriodic()
  {
  }

  @Override
  public void testInit()
  {
    CommandScheduler.getInstance().cancelAll();
    robotContainer.setDriveMode();
  }

  @Override
  public void testPeriodic()
  {
  }
  
  @Override
  public void simulationInit()
  {
  }

  @Override
  public void simulationPeriodic()
  {
    
  }
}
```