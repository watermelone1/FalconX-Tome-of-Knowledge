
Despite being about programming it is important to be aware of the electrical systems of the robot including connections and components
CAN
---
CAN wires green and yellow and are used to send signals between components including the RIO, and all motor controllers.

Ethernet
---
Ethernet connections allow for sending large amounts of data between devices. Devices primarily connect to the radio including the Limelight, RIO, and sometimes the laptop. When running the robot in the pits at a competition, you must connect to the RIO using Ethernet.

Power Wires
---
Usually black and red wires all throughout the robot. Provides power to the entire robot. There are different sized wires for different parts of the robot.

Encoder Wires
---
Seven multicolored wires coming out of a NEO motor. It sends signals to the SparkMAX which tells it its position and velocity and other details about the motor. Broken or damaged encoder wires can cause the motor to move erratically.

Digital Input-Output (DIO)
---
DIO wires connect a sensor to a port on the RIO. The code can access it using the `DigitalInput` class. Digital Inputs can only be on or off.

Pulse Width Modulation (PWM)
---
PWM connections are rarely used anymore, they are primarily used to connect the RIO to a CIM Motor. CIM motors do not have an integrated position encoder and can only be set to a speed.
