Battery
---
Provides power to the robot. 12V battery but runs between 12.7 and 13.5V with nothing connected, up to ~12.9 with load. You can check the status of the battery using the Battery Beak, a charge of 130% is good. When handling the battery it is important not to hold it by the wires, this damages the wires and the leads which can lead to bad connections and acid spills.

PDH
---
The PDH distributes power throughout the robot. It has ports for motors as well has other combinations of voltages. To access the ports on the PDH as well as most other devices, you place a screwdriver in the slot or on the buttons and push it down to open the wire slots.

VRM
---
The VRM is similar to the PDH but has more small options. This provides power to the Radio, and the Limelight.

RSL
---
The Robot Safety Light (RSL), is an indicator on the top of the robot that tells you the robot status. It has three modes. 
- Off: The robot is off and safe to touch
- Solid On: The robot is on but not enabled. Do not touch electrical components and verify before touching or making adjustments to mechanical parts
- Flashing On: The robot is on and enabled. Do not touch the robot while it is in this state.

Radio
---
The Radio allows you to connect to the RIO from the laptop. It has 4 Ethernet ports labelled DS, RIO, AUX1 and AUX2. The DS port connects to the Driver Station Laptop when not used wirelessly. The RIO port connects to the RoboRIO. The AUX 1 and 2 ports connect to additional devices such as the Limelight.

Motor Controllers
---
Motor Controllers are required to control most motors. They connect into the CAN loop, allowing them to communicate with the RIO. The RIO cannot directly send commands to the motors so they instead send a message to the controllers which translate it into electrical currents which run the motor. They also can record the position and velocity of the motor although are not accurate enough for precise control. As of 2025 we use REV NEO Motors which require an external REV SparkMax. CTRE Motors have integrated motor controllers.

Gyroscope
---
The gyroscope allows us to record accurate rotation and acceleration data. Different gyroscopes use different connections. As of 2025 we use the NavX 2 MXP, which plugs into the MXP port on the RIO or using Mini USB. The gyro is sometimes called the IMU (Inertial Measurement Unit)