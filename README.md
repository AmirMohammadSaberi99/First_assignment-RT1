### Table of Contents
- [What is the purpose?](#What-is-the-purpose?)
- [Installing and Running](#installing-and-running)
- [Exercise](#exercise)
- [Troubleshooting](#troubleshooting)
- [Robot API](#robot_api)
  - [Motors](#motors)
  - [Grabber](#grabber)
  - [Vision](#vision)
- [Coding](#coding)
  - [Drive Control](#drive-control)
  - [Turning Mechanism](#turning-mechanism)
  - [Locating Gold](#locating-gold)
  - [Gold Pickup](#gold-pickup)
  - [Gold Release](#gold-release)
- [Main Section](#main-section)


- [What is the purpose?](#What-is-the-purpose?)
  
The objective of the first assignment in the Research Track course is to provide a relative introduction to the Python programming language and its relevance to robotics. In this assignment, we aim to instruct the robot to collect boxes distributed throughout a two-dimensional space and place them in a designated area on the page.
These are pictures at the beginning and the end of the process.


At the first we can see this posture.

![bandicam 2023-11-05 13-50-31-576](https://github.com/Alansaberi/First_assignment-RT1/assets/64252685/02468429-a5df-4f77-8938-90514434a9e1)

And here's the final result.

![bandicam 2023-11-05 13-53-01-097](https://github.com/Alansaberi/First_assignment-RT1/assets/64252685/f25e2d28-baae-4611-8361-3fd71eb0fa81)


# Python Robotics Simulator

This is a simple, portable robot simulator developed by Student Robotics. Some of the arenas and the exercises have been modified for the Research Track I course.

### Installing and Running

The simulator requires a Python 2.7 installation, the pygame library, PyPyBox2D, and PyYAML.

Pygame, unfortunately, can be tricky (though not impossible) to install in virtual environments. If you are using pip, you might try `pip install hg+https://bitbucket.org/pygame/pygame`, or you could use your operating system's package manager. Windows users could use Portable Python. PyPyBox2D and PyYAML are more forgiving, and should install just fine using pip or easy_install.

### Troubleshooting

When running `python run.py <file>`, you may be presented with an error: `ImportError: No module named 'robot'`. This may be due to a conflict between `sr.tools` and `sr.robot`. To resolve, symlink `simulator/sr/robot` to the location of `sr.tools`.

On Ubuntu, this can be accomplished by:

1. Find the location of `sr.tools`: `pip show sr.tools`
2. Get the location. In my case, this was `/usr/local/lib/python2.7/dist-packages`
3. Create symlink: `ln -s path/to/simulator/sr/robot /usr/local/lib/python2.7/dist-packages/sr/`

### Exercise

To run one or more scripts in the simulator, use `run.py`, passing it the file names.

I am proposing you three exercises, with an increasing level of difficulty. The instruction for the three exercises can be found inside the `.py` files (`exercise1.py`, `exercise2.py`, `exercise3.py`).

When done, you can run the program with:

```shell
$ python run.py exercise1.py  

You also have the solutions of the exercises (folder solutions):
$ python run.py solutions/exercise1_solution.py

```
## Robot API
The API for controlling a simulated robot is designed to be as similar as possible to the SR API.

### Motors
The simulated robot has two motors configured for skid steering, connected to a two-output Motor Board. The left motor is connected to output 0 and the right motor to output 1.

The Motor Board API is identical to that of the SR API, except that motor boards cannot be addressed by serial number. So, to turn on the spot at one quarter of full power, one might write the following:

```shell
R.motors[0].m0.power = 25
R.motors[0].m1.power = -25
```

## Grabber
The robot is equipped with a grabber, capable of picking up a token which is in front of the robot and within 0.4 metres of the robot's centre. To pick up a token, call the R.grab method:
```shell
success = R.grab()
```

The R.grab function returns True if a token was successfully picked up, or False otherwise. If the robot is already holding a token, it will throw an AlreadyHoldingSomethingException.

To drop the token, call the R.release method.

Cable-tie flails are not implemented.

# Vision
To help the robot find tokens and navigate, each token has markers stuck to it, as does each wall. The R.see method returns a list of all the markers the robot can see, as Marker objects. The robot can only see markers which it is facing towards.

Each Marker object has the following attributes:

- info: a MarkerInfo object describing the marker itself. Has the following attributes:
  - code: the numeric code of the marker.
  - marker_type: the type of object the marker is attached to (either MARKER_TOKEN_GOLD, MARKER_TOKEN_SILVER, or MARKER_ARENA).
  - offset: offset of the numeric code of the marker from the lowest-numbered marker of its type. For example, token number 3 has the code 43, but offset 3.
  - size: the size that the marker would be in the real game, for compatibility with the SR API.
  - centre: the location of the marker in polar coordinates, as a PolarCoord object. Has the following attributes:
    - length: the distance from the centre of the robot to the object (in meters).
    - rot_y: rotation about the Y axis in degrees.
    - dist: an alias for centre.length.
  - res: the value of the res parameter of R.see, for compatibility with the SR API.
  - rot_y: an alias for centre.rot_y.
  - timestamp: the time at which the marker was seen (when R.see was called).

For example, the following code lists all of the markers the robot can see:

```shell
markers = R.see()
print "I can see", len(markers), "markers:"

for m in markers:
    if m.info.marker_type in (MARKER_TOKEN_GOLD, MARKER_TOKEN_SILVER):
        print " - Token {0} is {1} meters away".format( m.info.offset, m.dist )
    elif m.info.marker_type == MARKER_ARENA:
        print " - Arena marker {0} is {1} meters away".format( m.info.offset, m.dist )
```

```shell

Now, the text is formatted in GitHub-style with proper code blocks and headings.
```
# Here is the pseduocode for the program 
### we will break down every line in the next chapter.
```shell
Initialize Robot (R)
Set ORIENTATION_THRESHOLD and DISTANCE_THRESHOLD
first = 1
first_box_placed = True

while first is True:
    dist, rot_y = find_token_gold()
    
    if dist == -1:
        Print "I don't see any token!!"
        Set first to 0  # If no markers are detected, end the program
    else if dist < DISTANCE_THRESHOLD:
        Print "Found it!"
        R.grab()  # If close to the token, grab it
        Print "Gotcha!"
        Set first to 0
    else if -ORIENTATION_THRESHOLD <= rot_y <= ORIENTATION_THRESHOLD:
        Print "Ah, here we are!."
        Drive with speed 20 for 0.8 seconds
    else if rot_y < -ORIENTATION_THRESHOLD:
        Print "Left a bit..."
        Turn with speed -2 for 0.5 seconds
    else if rot_y > ORIENTATION_THRESHOLD:
        Print "Right a bit..."
        Turn with speed 2 for 0.5 seconds

Turn left with speed -8 for 1.5 seconds
Drive with speed 30 for 6 seconds
dist, rot_y = find_token_gold()
Release the grabbed token
Drive with speed -20 for 1.5 seconds
Turn with speed -10 for 4 seconds
Drive with speed 20 for 2 seconds

for each box in range(5):
    first_box_placed = True
    
    while first_box_placed is True:
        dist, rot_y = find_token_gold()

        if dist == -1:
            Print "I don't see any more tokens. All boxes are placed!"
            Turn with speed 10 for 1 second
        else if dist < DISTANCE_THRESHOLD:
            Print "Found it!"
            R.grab()  # If close to the token, grab it
            Print "Gotcha!"
            Turn with speed 20 for 2.5 seconds  # Adjust robot position for the next box placement
            Drive with speed 22 for 6 seconds  # Move to the reference position
            dist, rot_y = find_token_gold()
            Release the grabbed box
            Drive with speed -20 for 1.5 seconds  # Move away from the reference position
            Turn with speed -10 for 4 seconds
            Drive with speed 20 for 2 seconds
            Set first_box_placed to False  # Box placed, exit the loop
        else if -ORIENTATION_THRESHOLD <= rot_y <= ORIENTATION_THRESHOLD:
            Print "Ah, here we are!."
            Drive with speed 25 for 0.8 seconds
        else if rot_y < -ORIENTATION_THRESHOLD:
            Print "Left a bit..."
            Turn with speed -2 for 0.5 seconds
        else if rot_y > ORIENTATION_THRESHOLD:
            Print "Right a bit..."
            Turn with speed 2 for 0.5 seconds
```
