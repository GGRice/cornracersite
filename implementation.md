---
title: Project Implementation
layout: default
filename: implementation
---

# Implementation


## Architecture
We followed a standard Model View Controller (MVC) layout. The **Model** runs car physics, updates car position, and detects crashes. The **View** renders the background, foreground, and car. The **Controller** is split up into two parts, one for controlling the car and one for mouse input.

![UML_Diagram](/assets/UMLDiagram2.png)


## Model
The fundamental part of our racing software is our physics engine. The engine runs a realistic physics simulation of road-tire interactions. It is a two-tire model of a car, similar to a bike that can't fall over. The equations and constants of road-tire dynamics were selected from research papers on tire properties. The tire-road interaction can be simplified to two forces, as seen in the image.
![tire_model](/assets/tire_model.svg)

The blue arrow shows the back wheel accelerating the car. When the back wheel is powered by a engine, and pushes the car forward. The back wheel does not turn, so its normal force tries to keep the car going straight.
Because tires spin in only one direction, they apply a large normal force perpendicular to the direction of spin. When the front wheel turns, it applies a force towards the direction that it is pointing. This force is offset from the center of mass so it imparts a torque on the car, creating rotation.

Due to slip perpendicular to the tires, if the car is moving forward and turns the front wheel sharply, it looses grip with the surface and creates less force than it can. This is called understeer. Our model accounts for this. The controller does not allow a user with keyboard input understeer.
Oversteer occurs when the front tires grip the road more than the back and a drift or spinnout will occur. Because tire slip from acceleration is abstracted out to a constant, the car cannot drift, but oversteer still can occur.

The simulation models a virtual LIDAR sensor that uses raycasting to detect the car's distance from walls. The distance information from 20 rays is fed into the autonomous driver.

Finally, the model also detects crashes by drawing a rectangle around the car and detecting whether any points lie off of the road.


## Controller

The controller is split into two parts, one drives the car and the other controls mouse input for buttons and drawing. All input is done with Pygame.

#### Car Control
When a user selects to drive manually, they can control the car with arrow keys. Because the keyboard is limited to binary input, the steering angle changes proportionally to the velocity of the car. The faster it is moving, the less the 'steering wheel' rotates, because a large wheel angle at fast speeds would cause the car to be unstable (throwing it into over- or understeer). We tweaked the parameters to match a realistic driving scenario from our own experiences to make the controls feel good, but a keyboard does not match an actual steering wheel for input.
The second mode for controlling the car hands over the virtual steering wheel to an algorithm. Using information from the LIDAR, it can directly modify the steering wheel angle and accelerometer/brake to maneuver the car. This algorithm was created by evolving the weights each lidar had on the steering angle and accelerometer. [Read more about it here.](/evolution).

#### Mouse Interaction
There is also user interaction through a mouse interface. We created clickable buttons for navigating the menu and for starting a new track. The user can then use their mouse to draw a new custom track for them to drive on. After clicking "Draw Track" the current track is erased and the user can drag their mouse around to paint a new track. After releasing the mouse button, the new track is initialized with a car, and the world spawns adequate amounts of corn.

## View
To render all of this we have a View class.

Every time the frame updates, the View draws the ground, track, foreground (corn and cows), and car.

We wanted to have a lot of corn on the field, but it was very slow drawing each stalk individually for every frame. Instead, during initialization, it draws the corn and other foreground (cows and barn) onto a canvas and then only has to 'blip' this single image onto the world. It does the same thing to convert the road matrix into an image once, and can then just blip it onto the view.
It then draws the car at its current location and orientation.

The view can be disabled during training so that all computer processing can be dedicated to the physics simulation, speeding up trials.
View also handles playing sound effects and music.
