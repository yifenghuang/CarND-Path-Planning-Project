# Path Planning Project

## Rubric Points:
###The code compiles correctly.
Yes of course.
###The car is able to drive at least 4.32 miles without incident..
Yes, * Miles achieved.  
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

Collision identifier was introduced in my code block. sensor fusion data was injected into the algorithm, inside which we calculate other vehicle’s speed, s coordinate from Frenet and lane number. and using them for the collision
detection. 

we split the collision detection into 2 categories, if there is a car drive in front of us less than a safety distance, then we activate the too_close flag, set it to ‘true’. 

 
```
 if((check_car_lane!=cur_lane) && (check_car_s-car_s)<10.0 && (check_car_s-car_s)<30.0){

  too_close = true;

} 
```


if a car drives in adjacent lane, either drives in front of me up to 30 meters or later behind less than 10 meters, will trigger a specific flag labeling the path free of impact, then preparing for lane change.  

```
if((check_car_lane==cur_lane) && (car_s-check_car_s)>0.0 && (check_car_s-car_s)<30.0){
    ...
}
```

a Boolean vector is used to labeling/activate the available adjacent lane/lanes. 

```
vector<bool> laneFlag = {true,true,true};

laneFlag[cur_lane] = false; '
```

in order to labeling dis-qualified lanes:


```
if((check_car_lane==cur_lane) && (car_s-check_car_s)>0.0 && (check_car_s-car_s)<30.0){

    laneFlag[check_car_lane] = false;

}
```


Freedom of Lane changing maneuver is constrained so double lane slalom is prohibited. 

 
```
bool oneLaneChange(int cur_lane, int target_lane)
{   
if (cur_lane == 0 && target_lane == 1) {

return true;  
}   
if (cur_lane == 1 && (target_lane == 0 || target_lane == 2)) {

return true;   
}
if (cur_lane == 2 && target_lane == 1) {

return true;  
}
    return false;
}
```

 
to trigger lane shifting. if too_close, meanwhile exists a qualified candidate lane labeled ‘ture’, then activate lane shifting.  

 
```
if (too_close) {
   
   for (int i = 0; i < 3; i++) {

      if (laneFlag[i] && oneLaneChange(lane, i) {

             cur_lane = i;        
      } 
   }
}
```

 
To ensure the car drives according to the speed limit at 50 mph, the car lunched at 0 mph with step size of velocity coefficient at 0.224, which results a reasonable acceleration without introducing any uncomfortable jerk. the maximum
velocity is also under monitoring, if we go beyond 49.5mph, no more speed up.
too_close label is activated to reduce the speed by the same rate to avoid jerk feeling while decelerating. 

 
```
if (too_close) {

    ref_vel -= 0.224;

} else if (ref_vel < 49.5) {

    ref_vel += 0.224;
}
```

 
as taught in the class, trajectory generation module yield a smooth cubic spine. this track interpolation method provided an outperformed result. the spline library header file is quoted, and we feed 5 points into the pipeline: 2 points from the trajectory generated in the last time step; 3 more waypoints at 30/60/90 meters ahead in a given lane. 


```
vector<double> next_wp0 = getXY(car_s + 30/60/90, (2 + 4 * cur_lane), map_waypoints_s,map_waypoints_x, map_waypoints_y);
```


Finally there is a counter introduced. the purpose is to moderate the lane changing response, to discourage an aggressive driving
behavior. 

 
```
num_units_in_current_lane += 1;
   ...
if ((laneFlag[i] && oneLaneChange(lane,i) && (num_units_in_current_lane > 10)){
   ...
num_units_in_current_lane = 0;
}
```

