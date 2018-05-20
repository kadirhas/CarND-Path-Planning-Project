# CarND-Path-Planning-Project
Self-Driving Car Engineer Nanodegree Program solution

[//]: # (Image References)

[image1]: ./Docs/Avoid_collision.png "Avoiding collisions"
[image2]: ./Docs/Braking_with_traffic.png "The car brakes whenever there is no way to move around"
[image3]: ./Docs/Distance.png "Complete lap"
[image4]: ./Docs/Lane_change.png "The vehicle change lanes to avoid traffic"
[image5]: ./Docs/Staying_in_lanes.png "The vehicle stays in lanes"

### Simulator.
You can download the Term3 Simulator which contains the Path Planning Project from the [releases tab (https://github.com/udacity/self-driving-car-sim/releases/tag/T3_v1.2).

### Goals
In this project your goal is to safely navigate around a virtual highway with other traffic that is driving +-10 MPH of the 50 MPH speed limit. The car tries to go as close as possible to the 50 MPH speed limit, which means passing slower traffic when possible, while that other cars are trying to change lanes too. The car avoids hitting other cars at all cost as well as driving inside of the marked road lanes at all times, unless going from one lane to another. The car is able to make one complete loop around the 6946m highway. Since the car is trying to go 50 MPH, it should take a little over 5 minutes to complete 1 loop. Also the car should not experience total acceleration over 10 m/s^2 and jerk that is greater than 10 m/s^3.

## Basic Build Instructions

1. Clone this repo.
2. Make a build directory: `mkdir build && cd build`
3. Compile: `cmake .. && make`
4. Run it: `./path_planning`.

## Dependencies

* cmake >= 3.5
  * All OSes: [click here for installation instructions](https://cmake.org/install/)
* make >= 4.1
  * Linux: make is installed by default on most Linux distros
  * Mac: [install Xcode command line tools to get make](https://developer.apple.com/xcode/features/)
  * Windows: [Click here for installation instructions](http://gnuwin32.sourceforge.net/packages/make.htm)
* gcc/g++ >= 5.4
  * Linux: gcc / g++ is installed by default on most Linux distros
  * Mac: same deal as make - [install Xcode command line tools]((https://developer.apple.com/xcode/features/)
  * Windows: recommend using [MinGW](http://www.mingw.org/)
* [uWebSockets](https://github.com/uWebSockets/uWebSockets)
  * Run either `install-mac.sh` or `install-ubuntu.sh`.
  * If you install from source, checkout to commit `e94b6e1`, i.e.
    ```
    git clone https://github.com/uWebSockets/uWebSockets 
    cd uWebSockets
    git checkout e94b6e1
    ```

---

## Rubrics
### The code compiles correctly
The code can be compiled with Ubuntu 16.04 64-bit
### The car is able to drive at least 4.32 miles without incident
The vehicle can drive more than 4.32 most of the times, more than 1 laps. 

![alt text][image3]

### The car drives according to the speed limit

If there aren't any other vehicles on the same lane, the vehicle drives just under the speed limit, which can be seen on every other images.

### Max Acceleration and Jerk are not Exceeded
The vehicle does not violates the jerk and acceleration limits. All of the trajectories are calculated to provide maximum 5 m/s^2 or m/s^3 acceleration or jerk.

### Car does not have collisions

The car avoids collisions by changing lanes, or slowing down if there is no any other way.

![alt text][image2]

![alt text][image1]

### The car stays in its lane, except for the time between changing lanes

The car stays in its lane except to change lanes.

![alt text][image5]

### The car is able to change lanes

![alt text][image4]

## Model Documentation

The code for the path planning can be inspected in 3 parts: Scene understanding, Behavior planning, and Trajectory generation.
#### Scene understanding (main.cpp: lanes 264-298)
On this part, the sensor data is checked to understand if there is a vehicle on the same lane which might force to change lanes.
#### Behavior planning (main.cpp: lanes 299-378)
This is the part where the ego vehicle decides what to do. First, all of the occupied lanes are detected. If all of the lanes are full, lane change behavior is cancelled. If there is any empty lanes, then the distance of the current lane and the empty lane is considered. If the next lane is empty, it is chosen as the target lane, else the middle lane is also checked for occupancy. If the middle lane is full, lane change is cancelled. If the middle lane is also empty, the target lane is chosen as the middle lane to smooth the movement. Additional to all of this process, the velocity of the ego vehicle is adjusted according to the vehicle on the same lane to avoid any collisions.
#### Trajectory generation (main.cpp: lanes 379-480)
Trajectory generation part is responsible to generate a path which follows the target velocity and lane that is provided by the Behavior planning. Spline library is used to generate smooth paths for this part. First, the previous path points are obtained. Then, waypoints are obtained in front of the vehicle for the chosen lane. These points are transformed to ego vehicle coordinates to avoid numerical problems of the spline function. A spline is generated using the previous points and the new waypoints. Using the spline function, x and y points of the new trajectory is generated. The increments on the x points are chosen according to suit the reference velocity, which is provided by the behavior planner. The new generated points are transformed back to the earth coordinates and sent to the simulator.

## Discussion
The generated path planner is successful for most of the cases on the simulator. The planner fails on two conditions with the current settings:
* **Out of lane error:**
If the ego vehicle goes too slow, the generated path goes a little bit outside of the lane because of the generated spline shape, which happens rarely
* **Collision:** If another vehicle change lanes while the ego is also changing lanes, a collision happens. If the "switching lane" check is removed, this is avoided with a drawback: Then the path planner creates more aggresive paths to avoid collision, which causes to violate maximum acceleration or jerk limitations. This is also logical, because even with real drivers if the environment changes in an unexpected way, there is the cases which is not possible to avoid collision without making sudden movements. On this implementation, collisiions are chosen over jerk violations because of the occurance rate. 
This type of logical behavior planner is successful for this project only because of the simplicity of the simulation environment. For a more serious application, cost based method would perform better. 



