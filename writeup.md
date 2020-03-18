## Model Documentation

The simulator sends a number of things like Car's location, velocity, yaw rate, speed, frenet coordinates and sensor fusion data.  I implemented this project using a spline function, which fits a line to given points in a fairly smooth function.

### Prediction

The data from the sensor fusion and simulator is used to generate the prediction of what the other vehicles are likely to do. The sensor fusion data gives us data about the other cars nearby and we predict what they are likely to do. We change our behaviour on the basis of the Prediction of other cars behaviour

### Behaviour Planning 


My algo for behaviour planning is as follows-:

1. The car checks 30 m ahead if there is any car present or not.
2. If the senses a car 30 m ahead and it is slow moving so it tries to change the lane.
3. It tries to check the left lane first, if it free and contains no vehicle ahead of 30 m and 10 m behind the car. Behind because while changing lanes collision can happen with the cars moving point to point parallel.
4. If left lane is not available then it checks to switch to right lane and check if it free and contains no vehicle ahead of 30 m and 10 m behind the car.
5. If no option for lane change is there then it stays in the current lane without colliding with the car ahead.


### Trajectory Generation (Lines 415 - 516)

Our car calucates the trajectory based on its own speed, the speed of surrounding cars, current lane, intended lane and past points. To make trajectory smoother, we add last two points. If there are no previous points then we calculate the previous points from the current yaw and current car coordinates. Also the 3 points are added in next 30, 60, 90 meters to trajecotry. All these points are shifted to car reference angle.


### predicting and checking for other cars.

In the beginning i wrote seperate if block for each condition of possible collision, later my mentor gave me this idea to make it a function and call it whenever neccesary.
the final code in main.cpp uses this block many times to check for possible collisions before taking any action.
```
// function that returns if the car in the front of us is too close. Here we do not check any car coming from rear.


bool CheckIfCarIsTooClose(json sensor_fusion, int lane, double car_s, int prev_size, bool look_behind){
  for(int i=0;i<sensor_fusion.size();i++)
  {
    //cout<<"Inside Sesnor Fusion"<<endl;
    float d= sensor_fusion[i][6];
    if(d<(2+ 4*lane + 2) && d > (2+lane*4-2))
    {
      double vx= sensor_fusion[i][3];
      double vy= sensor_fusion[i][4];
      double check_speed = sqrt(vx*vx +vy*vy);
      double check_car_s = sensor_fusion[i][5];

      check_car_s += ((double)prev_size*0.02*check_speed);
   

      if(((check_car_s > car_s) && ((check_car_s - car_s) < 30)) || 
         (look_behind && (check_car_s < car_s) && ((car_s - check_car_s <10)))){
        return true;
      }
    }
  }
return false;
}
```