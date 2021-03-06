

## PID Controller

The PID controller for the self-driving car is a feedback mechanism where a cross track error (`CTE`) based on sensor meaurements from the car is input into the controller. The controller's job is to supply corrective feedback to the car in terms of a steering angle so that the car can use the output steering angle to adjust itself in order to stay on target and reduce or minimize the `CTE`. The PID controller has three components that make up the correction feedback function: the Proportional (`P`), the Integral (`I`), and the Derivative (`D`). The Proportional component can be viewed as the part that monitors the *Present*. `P` takes the *Present* CTE and makes a correction that is proportional to the CTE. On the other hand, the Integral part monitors the *Past*. `I` keeps track of all the *Past* CTEs that have been accumulated and provides the correction of an ongoing drift from the target. Finally, the Derivative component deals with the *Future*. `D` looks at the change in the CTE over a time period, which in turn gives it some insight into the *Future* CTE and makes a predictive correction based on this change in the CTE. In the correction update function, each of the three components are multiplied by their corresponding scaling factors and then summed up to output a correction angle that helps to steer the car back to the target track.

I implemented a PID controller class where a total error is calculated based on three initialization hyperparameters and a provided cross track error (the CTE) from the self-driving car simulator. The total error is iterpreted as a steering angle and is sent back to the simulator to change the steering angle of the car. As the simulator car is driven around the track, at periodic time intervals, the simulator sends the current CTE back to my client program. My client program then calculates the new steering angle and sends it back to the simulator...and on this goes, looping over and over until the car drives itself around the entire track.

The total error,or the steering angle, is calculated by taking the three hyperparameters, `Kp`, `Ki`, and `Kd` and applying them against the total error calculation in the form: `-Kp * p_error - ki * i_error - kd * d_error`. The `p_error` is the proportional error to the `CTE`. The `i_error` is the integral error composed of the `CTE` accumulated over time. Finally, the `d_error` is the error considered for the change in `CTE` over a time period. Each loop of the simulator communicating with my client program updates each error component and a new total error or steering angle is recalculated. As seen in the `PID::TotalError()` method each of the scaling factor hyperparameters are negated. This negation demonstrates that this total error output steering angle should be opposite of the input CTE and is meant as a corrective steering angle for the car so that the car steers itself back towards the target.

In order to get the car to drive all the way around the track, I needed to find the optimal values for `Kp`, `Ki`, and `Kd`. Typically, to accomplish this search for optimal values, a programmatic way, like the tweedle method, can be used. However, in this case, I manually, searched for these values. By trying out various values for the three hyperparameters, it became clear how each hyperparameter would affect the steering of the car. Observing that when Kd was too high, the car would over steer or over correct and causing the car to turn too much. In response, I would lower the `Kd` value in increments of 0.05 until the car seem to behave reasonably when negotiating turns. Similarily, the `Kp` value was adjusted based on obeserving how much in proprotion of the cars total displacement on the track is affected. Accordingly the `Kp` valued was tuned up or down. Finally, I tried changing the `Ki` value to see how the integral error term affects the steering of the car. Invariably, any value applied for this term contributed way too much to the steering and would cause the car to run off the track. Consequently, I kept the `Ki` value to zero. Below are the values tested through trial and error:

| Kp        | Ki        | Kd        | Comments                                                                                                            |
| --------: | --------: | --------: | :------------------------------------------------------------------------------------------------------------------ |
| 0.500     | 0.000     | 0.500     |                                                                                                                     |
| 0.750     | 0.000     | 0.250     |                                                                                                                     |
| 0.250     | 0.250     | 0.500     |                                                                                                                     |
| 0.750     | 0.500     | 0.150     |                                                                                                                     |
| 0.750     | 0.000     | 0.150     |                                                                                                                     |
| 0.500     | 0.000     | 0.000     |                                                                                                                     |
| 0.500     | 0.000     | 0.200     |                                                                                                                     |
| 0.750     | 0.000     | 0.200     |                                                                                                                     |
| 0.650     | 0.000     | 0.300     |                                                                                                                     |
| 0.650     | 0.000     | 0.150     |                                                                                                                     |
| 0.400     | 0.000     | 0.150     |                                                                                                                     |
| 0.400     | 0.000     | -0.150    |                                                                                                                     |
| 0.400     | 0.100     | 0.150     |                                                                                                                     |
| 0.400     | -0.100    | 0.150     |                                                                                                                     |
| 0.750     | -0.100    | 0.150     |                                                                                                                     |
| 0.750     | 0.000     | 0.015     |                                                                                                                     |
| 0.250     | 0.000     | 0.750     |                                                                                                                     |
| 0.350     | 0.000     | 0.750     |                                                                                                                     |
| 0.200     | 0.000     | 0.750     |                                                                                                                     |
| 0.200     | 0.000     | 0.800     |                                                                                                                     |
| 0.150     | 0.000     | 0.850     |                                                                                                                     |
| 0.100     | 0.000     | 0.900     |                                                                                                                     |
| 0.050     | 0.000     | 0.950     |                                                                                                                     |
| 0.050     | 0.000     | 1.000     |                                                                                                                     |
| 0.065     | 0.000     | 0.900     | car is going around the track with very slight weaving; some drifting over the right lane line on sharp left turns. |
| 0.075     | 0.000     | 0.900     |                                                                                                                     |
| 0.085     | 0.000     | 0.900     |                                                                                                                     |
| 0.080     | 0.000     | 0.900     |                                                                                                                     |
| 0.080     | 0.000     | 0.850     |                                                                                                                     |
| 0.100     | 0.000     | 0.850     |                                                                                                                     |
| 0.150     | 0.000     | 0.850     |                                                                                                                     |
| 0.125     | 0.000     | 0.825     |                                                                                                                     |
| 0.130     | 0.000     | 0.875     |                                                                                                                     |
| 0.080     | 0.000     | 0.825     |                                                                                                                     |
| 0.085     | 0.000     | 0.875     |                                                                                                                     |
| 0.090     | 0.000     | 0.875     |                                                                                                                     |
| 0.100     | 0.000     | 0.875     |                                                                                                                     |
| 0.100     | 0.000     | 0.900     |                                                                                                                     |
| 0.100     | 0.000     | 0.950     |                                                                                                                     |
| 0.100     | 0.000     | 1.000     |                                                                                                                     |
| 0.095     | 0.000     | 1.000     |                                                                                                                     |
| 0.085     | 0.000     | 1.000     |                                                                                                                     |
| 0.150     | 0.000     | 1.000     |                                                                                                                     |
| 0.150     | 0.000     | 1.100     |                                                                                                                     |
| 0.150     | 0.000     | 1.500     |                                                                                                                     |
| 0.200     | 0.000     | 1.500     |                                                                                                                     |
| 0.250     | 0.000     | 1.750     |                                                                                                                     |
| 0.250     | 0.000     | 1.250     |                                                                                                                     |
| 0.250     | 0.000     | 1.000     |                                                                                                                     |
| 0.200     | 0.000     | 1.000     |                                                                                                                     |
| 0.175     | 0.000     | 1.000     |                                                                                                                     |
| 0.150     | 0.000     | 1.000     |                                                                                                                     |
| 0.135     | 0.000     | 1.000     |                                                                                                                     |
| 0.125     | 0.000     | 1.000     |                                                                                                                     |
| **0.120** | **0.000** | **0.950** | fairly smooth; stays on track without going over lane lines; some oscillating behavior negotiating sharp turns      |