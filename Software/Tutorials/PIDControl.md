# Introduction to control
For a more in-depth explanation than what will be inside a relatively small markdown file, take *introduction to feedback systems (ELEC_ENG 360)*, *introduction to digital control (ELEC_ENG 374)*, *quadrotor design and control (COMP_SCI 410), introduction to mechatronics (MECH_ENG 333), or robotic manipulation (MECH_ENG 449)*

## Key concepts for today
- Control systems
- Open loop control
- Closed loop control
- PID control

## Control systems
This is kinda self-explanatory. A control system directs (or controls) the behavior of other devices or systems. Control systems can range from extremely simple to extremely complex. For the sake of simplicity, we will look at some very basic control systems/theory at a very high level for now.

## Open loop control
This is the most basic control system and what you might naively think of when it comes to control. Open loop control is when you just tell your system to "do this thing" and you just blindly let it go and attempt to "do that thing". Defined a little more formally, the directive the controller gives is independent of the output of the system that is being controlled. An example of an open loop controller is heating water to a set temperature by turning on a boiler for a set period of time. This system acts independently of what the water temperature actually ends up at, so while simple and very easy to implement, there is likely a large margin for error. For a more UROV-oriented example, say we want to turn the ROV 90 degrees to the right. In an open loop control system, we will simply drive the thrusters at a set power for a set amount of time without paying attention of where the robot is actually going.

## Closed loop control
This is a more robust system where the controller takes into account of the actual behavior of what it's controlling. Closed loop control is also known as feedback control because, well, we are getting feedback (and hopefully acting on it). Let's continue with the two examples from before. If we're heating water, we can read in our current temperature and adjust the power delivered to the heating element or the time we are turning it on. For the UROV, we could use an IMU to detect how much we've actually turned and then adjust how we're controlling the thruster from there.

## PID control
PID control is one of the most well-known and basic forms of feedback control. PID stands for proportional, integral, derivative. You can see what this means when we formally write out the control equation.

$$
u(t) = K_pe(t)+K_i\int e(t)dt + K_d\frac{d}{dt}e(t)
$$

where $u(t)$ is the controller's response, $e(t)$ is the error with respect to time, and $K_p$, $K_i$ and $K_d$ are the controller's proportional, integral, and derivative gains, respectively.

This still doesn't mean much, so let's break this down a little further. The most basic feedback controller we can work with is a P (proportional) controller (basically, we're setting the integral and derivative gains to 0). Mathematically, it looks like this

$$
u(t) = K_pe(t)
$$

So what we're doing here is driving our system with some proportional to its error to correct for it. Going back to the UROV example, if our desired angle is 90 degrees, but we are only at 50 degrees at this moment, we have an error of 40 degrees. So then we will drive our thrusters proportional to the 40 degree error.

The notable quirk with P control is overshoot. Your system will have overshoot, meaning we will go past our target and create error in the opposite direction. In the case of our UROV, let's say our controller ends up driving our thrusters too far so we're now at 100 degrees to the right instead of 90. So now we have a 10 degree error, just in the opposite direction as before. This error needs correction, so we're going to drive our thrusters proportional to this 10 degree error. Our corrective action might overshoot again, so now we have to make another correction, which could overshoot another time. This overshooting behavior might continue for a while, resulting in undesirable behavior called ringing.

To reduce ringing, we can add some damping to our system. We do this by adding the time derivative of the error to our control equation (PD control).

$$
u(t)=K_pe(t)+K_d\frac{d}{dt}e(t)
$$

So now our controller is also taking into account how fast our error is changing and damping the controller output accordingly so that way overshooting and ringing behavior isn't problematic.

The last problem we might deal with is steady state error that we can't get rid of (so our controller, even after settling will be stuck with some small error instead of 0). To fix this, we can integrate our error over time, which takes us back to the original pid control equation.

$$
u(t) = K_pe(t)+K_i\int e(t)dt + K_d\frac{d}{dt}e(t)
$$

Integrating the error over time means that we are taking into account the total error that we have across all error samples. So a small steady state error will lead to little movement in the P control to correct it, but if accumulated over time, the I term will dominate and then push to 0 steady state error.

Note that integral control, especially if $K_i$ is too large, can add instability to your controller, which ends up being way worse than having some mild steady state error.

## Example in code
This is what basic PID might look like in code. Assume that the gains are already defined.
```C
error_int += error; // integrate by just accumulating error
error_d = error_prev - error; // keep previous error term to get difference ("derivative") of error at this time
error_prev = error; // set previous error to current error so that we can calculate the next error derivative
control_output = Kp * error + Ki * error_int + Kd * error_d; // actual PID output
```
```
```
```
