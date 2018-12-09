This is a writeup of the Udacity Model Predictive Control project. I will go through the code in the 'src' folder and how it all helps to create a successful predictive controller.

The first step in "main.cpp" is to transform the waypoints and the initial state from map coordinates to vehicle coordinates. This is the format needed to plot trajectories in the simulation. Doing this upfront made it easy to solve the optimization problem and plot the trajectories in reference to the vehicle location. The initial state variables of px, py, and psi are all set to 0 as the location of the vehicle will always be the origin in vehicle coordinates and the facing angle will always be 0 degrees.

POINT_TRANSFORMATION IMG

Next I use 'polyfit()' to fit a 3rd order polynomial to the waypoints to represent the reference line. I then use this polynomial to calculate the initial CTE and EPSI.

POLYFIT IMG

I then deal with the 100ms latency. Since the actuations will be sent 100ms later than expected, I predict what the state will be 100ms in the future and use that as my initial state. This state is then sent to the MPC to solve the optimization problem and the first set of actuations are sent back to the vehicle to be executed before the loop runs again.

LATENCY IMG

Now I will go through the MPC code that actually solves the optimization problem. This code is found in "MPC.cpp".

First I set the 'N' and 'dt' variables determining the amount of timesteps in the future to project as well as the distance of these timesteps. I settled on the values 'N=10' and 'dt=0.1'. I experimented with roughly 4 different combinations of 'N' and 'dt' values: small N small dt, small N large dt, large N small dt, and large N large dt. I wanted to see what would happen with fairly extreme values. Here are a few short videos of how these different 'N' and 'dt' values affected the behavior of the vehicle:

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

I used 3 basic elements in my cost function. First I added the CTE and the EPSI values as well as a reference velocity of 100 to prevent the vehicle from stopping. I increased the multiplicative factor until the vehicle was able to make it around the track. Second I added the actuator values to minimize the use of the actuators. I didn't add too much of a multiplicative factor as I didn't want to penalize the use of actuators too much. Finally I added a penalty for having too big of a difference between successive actuator use. This helped smoothe the projected path.

COST_FUNCTION IMG



