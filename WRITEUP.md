This is a writeup of the Udacity Model Predictive Control project. I will go through the code in the 'src' folder and how it all helps to create a successful predictive controller.

The first step in "main.cpp" is to transform the waypoints and the initial state from map coordinates to vehicle coordinates. This is the format needed to plot trajectories in the simulation. Doing this upfront made it easy to solve the optimization problem and plot the trajectories in reference to the vehicle location. The initial state variables of px, py, and psi are all set to 0 as the location of the vehicle will always be the origin in vehicle coordinates and the facing angle will always be 0 degrees.

POINT_TRANSFORMATION IMG

Next I use 'polyfit()' to fit a 3rd order polynomial to the waypoints to represent the reference line. I then use this polynomial to calculate the initial CTE and EPSI.

POLYFIT IMG

I then deal with the 100ms latency. Since the actuations will be sent 100ms later than expected, I predict what the state will be 100ms in the future and use that as my initial state. This state is then sent to the MPC to solve the optimization problem and the first set of actuations are sent back to the vehicle to be executed before the loop runs again.

LATENCY IMG

Now I will go through the MPC code that actually solves the optimization problem. This code is found in "MPC.cpp".

First I set the 'N' and 'dt' variables determining the amount of timesteps in the future to project as well as the distance of these timesteps. I settled on the values 'N=10' and 'dt=0.1'. I primarily experimented with the N values. When I went very small, N=2, the vehicle did not move very much. Here is a short video of the results:
