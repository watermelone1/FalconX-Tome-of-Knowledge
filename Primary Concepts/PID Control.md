A PID Controller (Proportional - Integral - Derivative) is a way to automatically move and manage a value such as position, or rotation, or distance, and adjust to keep it in place by using feedback.

## Theory

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

These together allow you to react to different scenarios in, all in one controller. The next problem comes with finding the constants for P I and D. There is no easy way to find these and the optimal numbers may be extremely small or extremely large. In general you start by finding P.

Set $K_i$ and $K_d$ to 0 and choose an arbitrary constant for $K_p$ . If you overshoot your target then lower P, if you undershoot then raise P. Continue until it reaches the target.

Next tune $K_d$ . When D is too high, you will oscillate around the target, too low and you won't get to the target quick enough. D also helps react to a moving target.

$K_i$ is a weird one. In a lot of cases it is fine to leave it at 0, however you may want to adjust it if you have a long distance or it takes a while to reach the target. A high value of I can reach the target faster, but overshoot more, a low value will can slow down to the target faster but oscillate more.






https://upload.wikimedia.org/wikipedia/commons/3/33/PID_Compensation_Animated.gif

## Programming