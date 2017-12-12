# Path Planning Project

## Rubric Points:
###The code compiles correctly.
Yes of course.
###The car is able to drive at least 4.32 miles without incident..
Yes, 43.38 Miles achieved.  
<img src="./screenst.png">
###The car drives according to the speed limit.
Yes. I set the speed limit to 49.5 MPH.
###Max Acceleration and jerk are not Exceeded.
Maximum acceleration value is constrained. Meanwhile the double lane changing is forbidened so that the lateral jerk is under control. 
###Car dose not have collisions
Yes.
###The car stays in its lane for the time between changing lanes. 
Yes. The lane changing is very neat.
###The car is able to change lanes
Yes.
### The car is able to smoothly change 
If the car meet a slow car when there is a lane available at a nearby lane, it will trigger the lane changing so that the deceleration is no longer need.
###There is a reflection on how to generate paths.
My implementation is based on the video presented in the class. 

Collision identifier is in the code block. sensor fusion data is already in the algorithm, which we calculate other vehicle’s speed, s coordinate and lane number for the collision detection algorithm. 

the collision detection have 2 parts, if there is a car drive in front of us less than a safety distance, then we set the too_close flag to ‘true’. 

 
```
 if((check_car_lane!=cur_lane) && (check_car_s-car_s)<10.0 && (check_car_s-car_s)<30.0)
 {
  too_close = true;
} 
```


if a car drives in adjacent lane and there is a drive in front of me up to 30 meters or behind me less than 10 meters will trigger a flag to preparing for lane change.  

```
if((check_car_lane==cur_lane) && (car_s-check_car_s)>0.0 && (check_car_s-car_s)<30.0)
{
    ...
}
```

a Bool vector is used to activate the available adjacent lanes. 

```
vector<bool> laneFlag = {true,true,true};

laneFlag[cur_lane] = false; '
```

to labeling dis-qualified lanes:


```
if((check_car_lane==cur_lane) && (car_s-check_car_s)>0.0 && (check_car_s-car_s)<30.0)
{
    laneFlag[check_car_lane] = false;
}
```


double lane change is prohibited:
```
bool if2lane(int cur_lane, int target_lane)
{
    if (cur_lane == 0 && target_lane == 1) return true;
    if (cur_lane == 1 && (target_lane == 0 || target_lane == 2)) return true;
    if (cur_lane == 2 && target_lane == 1) return true;
    return false;
}
```

 
to trigger lane shifting. if too_close, meanwhile exists a qualified candidate lane labeled ‘ture’, then activate lane shifting.  

 
```
if (too_close) 
{
   for (int i = 0; i < 3; i++) 
   {
      if (laneFlag[i] && oneLaneChange(lane, i) 
      {
             cur_lane = i;        
      } 
   }
}
```

 
the car lunched at 0 mph with step size of velocity coefficient at 0.224, can make a reasonable acceleration without introducing any uncomfortable jerk. 

the car have to drives according to the speed limit at 50 mph. So the maximum
velocity is limited, if we go beyond 49.5mph, no more speed up.
too_close label is activated to reduce the speed by the same rate to avoid jerk feeling while decelerating. 

 
```
if (too_close) 
{
    ref_vel -= 0.224;
} 
else if (ref_vel < 49.5) 
{
    ref_vel += 0.224;
}
```

 
Trajectory generation module need a smooth cubic spine.I use the spline library header file. I feed 2 points in the last time step and 3 waypoints at 30/60/90 meters.


```
vector<double> next_wp0 = getXY(car_s + 30/60/90, (2 + 4 * cur_lane), map_waypoints_s,map_waypoints_x, map_waypoints_y);
```
