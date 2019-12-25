##PID Control
PID controllers are simple reactive controllers that are widely used. The difference between the measured and the desired value (setpoint) of a process variable of a system is fed into the PID controller as an error signal. Depending on the PID parameters a control output is generated to steer the system closer to the setpoint. In the present project, a car simulator produces the error signal as the distance between the actual car position on the road and a reference trajectory, known as cross-track error (cte). The PID controller is designed to minimize the distance to this reference trajectory. The primary control output of the PID controller here is the steering angle.

###P - proportional gain
The proportional term computes an output proportional to the cross-track error. A pure P - controller is unstable and at best oscillates about the setpoint. The proportional gain contributes a control output to the steering angle of the form -K_p cte with a positive constant K_p.

###D - differential gain
The oscillations caused by purely D control can be mitigated by a term proportional to the derivative of the cross-track error. The derivative gain contributes a control output of the form -K_d d/dt cte, with a positive constant K_d.

###I - integral gain
A third contribution is given by the integral gain which simply sums up the cross-track error over time. The corresponding contribution to the steering angle is given by -K_i sum(cte). Thereby, biases can be mitigated, for instance if a zero steering angle does not correspond to a straight trajectory. At high speeds this term can also be useful to accumulate a large error signal quickly, for instance when the car is carried out sideways from the reference trajectory. This allows to reduce proportional gain, which causes oscillations at high speeds. It is also beneficial to limit the memory of this term to avoid overshooting. Here, we used an exponentially weighted moving average for this purpose.

##Extensions
###Linear parameter-varying control
Using a single PID controller can result in suboptimal performance at different speeds. We therefore implemented the possibility to accomodate different setpoints and to vary the parameters linearly according to the speed the car. For example the coefficient K_i is increased linearly as a function of the speed of the car: K_i = alpha_i * v.

###Smoothened steering angle
The output of the PID controller can result in jerky steering, causing the car to oscillate. A simple heuristic measure to mitigate this instability was to simply use a weighted average of the previous and the current steering angle. While this measure reduces reactivity, it makes the ride much smoother and allows to achieve higher speeds.

###Emergency braking
In case the cross-track error increase too quickly a simple emergency measure was to reduce throttle to zero for a few iterations. More severe measures like braking can also be implemented in the same way and lead to reduced cross-track errors at the cost of reduced speed.

###Hyperparameter Tuning
All parameters were tuned manually. The reason for this approach was that slightly wrong parameters quickly lead to unrecoverable crashes of the car and require a manual restart of the simulator. A more systematic and algorithmic approach including automatic restarts of the simulator can be taken, but here I decided to take the pedestrian approach, because it provides an intuitive understanding of thethe importance of the different contributions. The algorithm used was roughly as follows.

single P controller for one setpoint, with K_i and K_d = 0
increase K_d until oscillations subside.
in case of crashes: find cause. a) if slow reactivity is the cause -> reduce steering angle smoothing, increase K_p or K_i b) if oscillations are the cause -> reduce K_p, K_i or brake
Rounds topping out at speeds of 95mph can be acheived this way, but a much safer ride is obtained when the target speed is set to 70mph. This is what is shown in the video linked below.
