---
title: Project Implementation
layout: default
filename: implementation
---

# Implementation


## Architecture
We followed a standard Model View Controller (MVC) layout. The **Model** runs car physics, updates car position, and detects crashes. The **View** renders the background, foreground, and car. The **Controller** is split up into two parts, one for controlling the car and one for clicking buttons and drawing new track.

![UML_Diagram](/assets/UMLDiagram2.png)

### Model
The model runs a realistic physics simulation of a two tire model, similar to a bike that can't fall over. Tire - road dynamics are modeled, and realistic constants were selected from research papers on tire dynamics. The tire-road interaction can be easily explained as two forces, as seen in the image. The two forces are perpendicular to each other.
![tire_model](/assets/tire)


The back wheel applies a force based on user input, modeled from how much power a car can give.
Because tires spin in only one direction, they apply a large force perpendicular to the direction of spin. When the front wheel turns, it doesn't rotate the car automatically, but applies a force orthogonal to the direction of motion. This force then rotates the car. Due to slip perpendicular to the tires, if the car is moving forward and turns the front wheel drastically, it looses grip with the surface and turns less than ideal. This is called understeer. Oversteer occurs when the front tires grip the road more than the back and a drift or spinnout will occur.

Because tire slip due to acceleration is abstracted out to a constant, the car cannot drift, but it does understeer.

Finally, the model creates a virtual LIDAR sensor that uses raycasting to detect the car's distance from walls. It shoots $20$ equally spaced beams around itself. This information is then fed into an algorithm that can drive the car autonomously.

### Controller

The controller is split into two parts, one drives the car and the other controls mouse input, buttons and drawing. All input is done with Pygame.

#### Car Control
When a user selects to be the driver, they can control the car with arrow keys. Because the keyboard is limited to binary input, the steering angle changes proportionally to the velocity of the car. The faster it is moving, the less the 'steering wheel' rotates, because a large wheel angle at fast speeds would cause the car to be unstable (throwing it into over- or understeer). We tweaked the parameters to match a realistic driving scenario from our own experiences to make the controls feel good, but a keyboard does not match an actual steering wheel for input.
The second mode for controlling the car hands over the virtual steering wheel to an algorithm. Using information from the LIDAR, it can directly modify the steering wheel angle and accelerometer/brake to maneuver the car. This gives it a higher precision input.

#### Mouse Interaction
There is also user interaction through a mouse interface. We created clickable buttons for navigating the menu and for starting a new track. The user also uses their mouse for drawing a new custom track for them to drive on. After clicking "Draw Track" the track is erased and the user can paint a track onto the world by dragging their mouse. After releasing the mouse button, the new track is initialized a car and the world spawns corn.

## View
To see all of this we have a View.

Every frame it draws the ground, track, foreground (corn and cows), and car.

We wanted to have a lot of corn on the field, but it was very slow drawing individual stalks every frame. Instead, during initialization it draws the foreground (the corn, cows, and barn) onto a transparent canvas and then only has to blip one image onto the world. It does the same thing to draw the road to speed up frame drawing.
It then draws the car at its current location and direction.
The view can be disabled during training so that all computer processing can be dedicated to the physics simulation, speeding up trials.
View also handles playing sound effects and music.
