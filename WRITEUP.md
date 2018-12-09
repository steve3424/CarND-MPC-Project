This is a writeup of the Udacity Model Predictive Control project. I will go through the code in the 'src' folder and how it all helps to create a successful predictive controller.

The first step in "main.cpp" is to transform the waypoints and the initial state from map coordinates to vehicle coordinates. This is the format needed to plot trajectories in the simulation. Doing this upfront made it easy to solve the optimization problem and plot the trajectories in reference to the vehicle location. The initial state variables of px, py, and psi are all set to 0 as the location of the vehicle will always be the origin in vehicle coordinates and the facing angle will always be 0 degrees.


![Alt text](images/point_transformation.png?raw=True)

Next I use 'polyfit()' to fit a 3rd order polynomial to the waypoints to represent the reference line. I then use this polynomial to calculate the initial CTE and EPSI.

![Alt text](images/polyfit.png?raw=True)

I then deal with the 100ms latency. Since the actuations will be executed 100ms later than expected, I predict what the state will be 100ms in the future and use that as my initial state. This state is then sent to the MPC to solve the optimization problem and the first set of actuations are sent back to the vehicle to be executed before the loop runs again.

![Alt text](images/latency.png?raw=True)

Now I will go through the MPC code that actually solves the optimization problem. This code is found in "MPC.cpp".

First I set the 'N' and 'dt' variables which determine how far in the future to predict the path. I settled on the values 'N=10' and 'dt=0.1'. I experimented with roughly 4 different combinations of 'N' and 'dt' values: small N small dt, small N large dt, large N small dt, and large N large dt. I wanted to see what would happen with fairly extreme values and work from there. Here are a few short videos of how these different 'N' and 'dt' values affected the behavior of the vehicle:

N = 2, dt = 0.01
SMALL-SMALL VIDEO

N = 2, dt = 2.0
SMALL-LARGE VIDEO

N = 30, dt = 0.1
LARGE-SMALL VIDEO

N = 30, dt = 3.0
LARGE-LARGE VIDEO

And here is what happened with the final values that I settled on:

N = 10, dt = 0.1
GOLDILOCKS VIDEO

I am not entirely sure why these particular 'N' and 'dt' values caused the behavior that they did. I suspect that it has something to do with the cost function, but I will have to experiment further to fully understand why they have the affect that they did. 

I used 3 basic elements in my cost function. First I added the CTE and the EPSI values as well as a reference velocity of 100 to prevent the vehicle from stopping. I increased the multiplicative factor until the vehicle was able to make it around the track. Second I added the actuator values to minimize the use of the actuators. I didn't add too much of a multiplicative factor as I didn't want to penalize the use of actuators too much. Finally I added a penalty for having too big of a difference between successive actuator use. This helped smoothe the projected path.

![Alt text](images/cost_function.png?raw=True)


Here I set the bounds and constraints of the model. Since the 'vars' vector is where the solution of the optimization will be the state values are unbounded while the actuator values are bounded. The delta output will only be between -25 and 25 degrees while the acceleration will be between -1 and 1.

![Alt text](images/var_bounds.png?raw=True)

The constraints are where we implement the vehicle model. We set the initial state indices to the initial state values as those will not be changed. The rest we set to 0 in order later implement the kinematic model.

Here I implemented the kinematic model with the constraints. This ensures that each successive state variable is exactly equal to the calculation of the kinematic equations.

![Alt text](images/kinematic_model.png?raw=True)

