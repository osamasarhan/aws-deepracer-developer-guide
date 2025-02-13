# Input Parameters of the AWS DeepRacer Reward Function<a name="deepracer-reward-function-input"></a>

The AWS DeepRacer reward function takes a dictionary object as the input\. 

```
def reward_function(params) :
    
    reward = ...

    return float(reward)
```

The `params` dictionary object contains the following key\-value pairs:

```
{
    "[all\_wheels\_on\_track](#reward-function-input-all_wheels_on_track)": Boolean,    # flag to indicate if the vehicle is on the track
    "[x](#reward-function-input-x_y)": float,                        # vehicle's x-coordinate in meters
    "[y](#reward-function-input-x_y)": float,                        # vehicle's y-coordinate in meters
    "[distance\_from\_center](#reward-function-input-distance_from_center)": float,     # distance in meters from the track center 
    "[is\_left\_of\_center](#reward-function-input-is_left_of_center)": Boolean,      # Flag to indicate if the vehicle is on the left side to the track center or not. 
    "[heading](#reward-function-input-heading)": float,                  # vehicle's yaw in degrees
    "[progress](#reward-function-input-progress)": float,                 # percentage of track completed
    "[steps](#reward-function-input-steps)": int,                      # number steps completed
    "[speed](#reward-function-input-speed)": float,                    # vehicle's speed in meters per second (m/s)
    "[steering\_angle](#reward-function-input-steering_angle)": float,           # vehicle's steering angle in degrees
    "[track\_width](#reward-function-input-track_width)": float,              # width of the track
    "[waypoints](#reward-function-input-waypoints)": [[float, float], … ], # list of [x,y] as milestones along the track center
    "[closest\_waypoints](#reward-function-input-closest_waypoints)": [int, int]    # indices of the two nearest waypoints.
}
```

The more detailed technical reference for the input parameters is as follows\. 

## all\_wheels\_on\_track<a name="reward-function-input-all_wheels_on_track"></a>

**Type: ** `Boolean`

**Range: ** `True|False`

A `Boolean` flag to indicate whether the vehicle is on\-track or off\-track\. It's off\-track \(`False`\) if any of its wheels are outside of the track borders\. It's on\-track \(`True`\) if all of the wheels are inside the two track borders\. The following illustration shows that the vehicle is on\-track\. 

![\[\]](http://docs.aws.amazon.com/deepracer/latest/developerguide/images/deepracer-reward-function-input-all_wheels_on_track-true.png)

The following illustration shows that the vehicle is off\-track\.

![\[\]](http://docs.aws.amazon.com/deepracer/latest/developerguide/images/deepracer-reward-function-input-all_wheels_on_track-false.png)

**Example: ** *A reward function using the `all_wheels_on_track` parameter*

```
define reward_function(params):
    #############################################################################
    '''
    Example of using all_wheels_on_track and speed
    '''

    # Read input variables
    all_wheels_on_track = params['all_wheels_on_track']
    speed = params['speed']

    # Set the speed threshold based your action space 
    SPEED_THRESHOLD = 1.0 

    if not all_wheels_on_track:
        # Penalize if the car goes off track
        reward = 1e-3
    elif speed < SPEED_THRESHOLD:
        # Penalize if the car goes too slow
        reward = 0.5
    else:
        # High reward if the car stays on track and goes fast
        reward = 1.0

    return reward
```

## closest\_waypoints<a name="reward-function-input-closest_waypoints"></a>

**Type**: `[int, int]`

**Range**: `[0:Max-2,1:Max-1]`

The zero\-based indices of the two neighboring `waypoint`s closest to the vehicle's current position of `(x, y)`\. The distance is measured by the Euclidean distance from the center of the vehicle\. `Max` is the length of the waypoints list\. In the illustration shown in [waypoints](#reward-function-input-waypoints), the `closest_waypoints` would be `[16, 17]`\. Another possibility could be `[16, 15]`\.

**Example**: A reward function using the `closest_waypoints` parameter\.

The following example reward function demonstrates how to use `waypoints` and `closest_waypoints` as well as `heading` to calculate immediate rewards\.

```
def reward_function(params):
    ###############################################################################
    '''
    Example of using waypoints and heading to make the car in the right direction
    '''

    import math

    # Read input variables
    waypoints = params['waypoints']
    closest_waypoints = params['closest_waypoints']
    heading = params['heading']

    # Initialize the reward with typical value 
    reward = 1.0

    # Calculate the direction of the center line based on the closest waypoints
    next_point = waypoints[closest_waypoints[1]]
    prev_point = waypoints[closest_waypoints[0]]

    # Calculate the direction in radius, arctan2(dy, dx), the result is (-pi, pi) in radians
    track_direction = math.atan2(next_point[1] - prev_point[1], next_point[0] - prev_point[0]) 
    # Convert to degree
    track_direction = math.degrees(track_direction)

    # Calculate the difference between the track direction and the heading direction of the car
    direction_diff = abs(track_direction - heading)
    if direction_diff > 180:
        direction_diff = 360 - direction_diff

    # Penalize the reward if the difference is too large
    DIRECTION_THRESHOLD = 10.0
    if direction_diff > DIRECTION_THRESHOLD:
        reward *= 0.5

    return reward
```

## distance\_from\_center<a name="reward-function-input-distance_from_center"></a>

**Type**: `float`

**Range**: `0:~track_width/2`

Displacement, in meters, between the vehicle center and the track center\. The observable maximum displacement occurs when any of the agent's wheels are outside a track border and, depending on the width of the track border, can be slightly smaller or larger than half the `track_width`\.

![\[\]](http://docs.aws.amazon.com/deepracer/latest/developerguide/images/deepracer-reward-function-input-distance_from_center.png)

**Example:** *A reward function using the `distance_from_center` parameter*

```
def reward_function(params):
    #################################################################################
    '''
    Example of using distance from the center 
    ''' 

    # Read input variable
    track_width = params['track_width']
    distance_from_center = params['distance_from_center']

    # Penalize if the car is too far away from the center
    marker_1 = 0.1 * track_width
    marker_2 = 0.5 * track_width

    if distance_from_center <= marker_1:
        reward = 1.0
    elif distance_from_center <= marker_2:
        reward = 0.5
    else:
        reward = 1e-3  # likely crashed/ close to off track

    return reward
```

## heading<a name="reward-function-input-heading"></a>

**Type**: `float`

**Range**: `-180:+180`

Heading direction, in degrees, of the vehicle with respect to the x\-axis of the coordinate system\.

![\[\]](http://docs.aws.amazon.com/deepracer/latest/developerguide/images/deepracer-reward-function-input-heading.png)

**Example:** *A reward function using the `heading` parameter*

For more information, see [`closest_waypoints`](#reward-function-input-closest_waypoints)\.

## is\_left\_of\_center<a name="reward-function-input-is_left_of_center"></a>

**Type**: `Boolean`

**Range**: `True | False`

A `Boolean` flag to indicate if the vehicle is on the left side to the track center \(`True`\) or on the right side \(`False`\)\. 

## progress<a name="reward-function-input-progress"></a>

**Type**: `float`

**Range**: `0:100`

Percentage of track completed\.

**Example:** *A reward function using the `progress` parameter*

For more information, see [steps](#reward-function-input-steps)\.

## speed<a name="reward-function-input-speed"></a>

**Type**: `float`

**Range**: `0.0:5.0`

The observed speed of the vehicle, in meters per second \(m/s\)\.

![\[\]](http://docs.aws.amazon.com/deepracer/latest/developerguide/images/deepracer-reward-function-input-speed.png)

**Example:** *A reward function using the `speed` parameter*

For more information, see [all\_wheels\_on\_track](#reward-function-input-all_wheels_on_track)\.

## steering\_angle<a name="reward-function-input-steering_angle"></a>

**Type**: `float`

**Range**: `-30:30`

Steering angle, in degrees, of the front wheels from the center line of the vehicle\. The negative sign \(\-\) means steering to the right and the positive \(\+\) sign means steering to the left\. The vehicle center line is not necessarily parallel with the track center line as is shown in the following illustration\.

![\[\]](http://docs.aws.amazon.com/deepracer/latest/developerguide/images/deepracer-reward-function-steering.png)

**Example:** *A reward function using the `steering_angle` parameter*

```
def reward_function(params):
    '''
    Example of using steering angle
    '''

    # Read input variable
    steering = abs(params['steering_angle']) # We don't care whether it is left or right steering

    # Initialize the reward with typical value 
    reward = 1.0

    # Penalize if car steer too much to prevent zigzag
    STEERING_THRESHOLD = 20.0
    if steering > ABS_STEERING_THRESHOLD:
        reward *= 0.8

    return reward
```

## steps<a name="reward-function-input-steps"></a>

**Type**: `int`

**Range**: `0:Nstep`

Number of steps completed\. A step corresponds to an action taken by the vehicle following the current policy\.

**Example:** *A reward function using the `steps` parameter*

```
def reward_function(params):
    #############################################################################
    '''
    Example of using steps and progress
    '''

    # Read input variable
    steps = params['steps']
    progress = params['progress']
    
    # Total num of steps we want the car to finish the lap, it will vary depends on the track length
    TOTAL_NUM_STEPS = 300

    # Initialize the reward with typical value 
    reward = 1.0

    # Give additional reward if the car pass every 100 steps faster than expected 
    if (steps % 100) == 0 and progress > (steps / TOTAL_NUM_STEPS) * 100 :
        reward += 10.0

    return reward
```

## track\_width<a name="reward-function-input-track_width"></a>

**Type**: `float`

**Range**: `0:Dtrack`

Track width in meters\.

![\[\]](http://docs.aws.amazon.com/deepracer/latest/developerguide/images/deepracer-reward-function-input-track_width.png)

**Example:** *A reward function using the `track_width` parameter*

```
def reward_function(params):
    #############################################################################
    '''
    Example of using track width
    '''

    # Read input variable
    track_width = params['track_width']
    distance_from_center = params['distance_from_center']

    # Calculate the distance from each border
    distance_from_border = 0.5 * track_width - distance_from_center

    # Reward higher if the car stays inside the track borders
    if distance_from_border >= 0.05:
        reward *= 1.0
    else:
        reward = 1e-3 # Low reward if too close to the border or goes off the track

    return reward
```

## x, y<a name="reward-function-input-x_y"></a>

**Type**: `float`

**Range**: `0:N`

Location, in meters, of the vehicle center along the x and y axes, of the simulated environment containing the track\. The origin is at the lower\-left corner of the simulated environment\.

![\[\]](http://docs.aws.amazon.com/deepracer/latest/developerguide/images/deepracer-reward-function-input-x-y.png)

## waypoints<a name="reward-function-input-waypoints"></a>

**Type**: `list` of `[float, float]`

**Range**: `[[xw,0,yw,0] … [xw,Max-1, yw,Max-1]]`

An ordered list of track\-depedent `Max` milestones along the track center\. Each milestone is described by a coordinate of \(xw,i, yw,i\)\. For a looped track, the first and last waypoints are the same\. For a straight or other non\-looped track, the first and last waypoints are different\.

![\[\]](http://docs.aws.amazon.com/deepracer/latest/developerguide/images/deepracer-reward-function-input-waypoints.png)

**Example** *A reward function using the `waypoints` parameter*

For more information, see [`closest_waypoints`](#reward-function-input-closest_waypoints)\.