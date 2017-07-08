

### project description
The goal of the project is to provide actuator commands to the simulator that successfully steers the vehicle along the track. The immediate estimated track path -waypoints for the coming stretch of track.- is provided through the websocket in the simulator along with the telemetry (position, velocity, heading).

### cost function and constraints
Cost function is defined in terms of CTE and PSI errors, in addition to the magnitude of actuators as well as the delta between sequential actuations to ensure smooth transitions between states. The importance of these three components is determined by their respective weights. Hence, I spent a bit time to get to the optimal weights to make the vehicle drive around with no issues. The final version of the cost function pays a lot of attention to CTE and PSI errors.

Secondly, we define the vehicle model that defines the state transitions. The kinematic model is defined in terms of constraints. Both the cost function and the constraints are defined in the `MPC.operator` method. The kinematic model is defined as follows:

The model state is:
```
 px: x-position of the vehicle
 py: y-position of the vehicle
 psi: Orientation of the vehicle in radians
 v: Velocity of the vehicle
```
The actuators of the vehicle are:
```
 delta_psi: Steering angle
 a: Acceleration
```
The update equations for our model used to predict future states are:
```
 px(t+1) = px(t) + v(t) * cos(psi(t)) * dt
 py(t+1) = py(t) + v(t) * sin(psi(t)) * dt
 psi(t+1) = psi(t) + v(t) / Lf * delta_psi * dt
 v(t+1) = v(t) + a * dt;
```


## MPC process
The algorithm fits a third degree polynomial to the incoming waypoints, which in turn is fed into the optimizer in `MPC.solver` to determine the steering and throttle variables that minimize the cost function over time steps. Number of time steps and the interval are hyper-parameters to the model and present a tradeoff. N = 20 and dt = 0.2 are chosen empirically and robust to high speeds. At each step, the process is repeated. Only the actuator parameters from the first time steps are kept and rest are discarded. The server passes the location information in map coordinates. Like in the previous projects, converting the map coordinates to vehicle coordinates to make the calculations much easier.

## Simulating latency
Latency is a reality of real-time systems. 
