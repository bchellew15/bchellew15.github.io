---
layout: post
title:  "Rubik's Cube Simulator (Python)"
date:   2024-11-16 06:22:45 -0700
categories: jekyll update
---

[The code for this project is [here][puzzle-code-repo], written in Python.]

I made a Rubik's cube simulator in Python. I used matplotlib to display and animate the cube because I'm already familiar with matplotlib for plotting.

# GUI

Here is a screenshot of the simulator:

![screenshot]({{ site.baseurl }}/images/rubiks_screenshot.png){: width="200"}

I used the matplotlib GUI, calling `plt.figure()` and plt.show().
A benefit is that I can click and drag to view the cube from different angles, even while animations are running, with no extra work on my part.
The buttons in the bottom left corner turn the cube faces. They are labeled using standard Rubik's cube notation. F is the front face, R is right, U is up, B is back, L is left, and D is down. The default turn direction is clockwise, and prime (') represents a counter-clockwise turn, so clicking the R' button turns the right face counter-clockwise.

Here is a demonstration video:
<video width="320" height="240" controls loop="" muted="" autoplay="">
   <source src="https://github.com/bchellew15/bchellew15.github.io/raw/refs/heads/main/images/rubiks_screen_record_480.mp4">
</video>

(edit) All (these functions) assume that other face rotations are complete before they start. When a button is clicked, the (corresponding function) is added to a queue, so that any pending face rotations can be completed.

The buttons in the top left corner are for high-level functions. The "reset" button just resets the colors. The "randomize" button executes 20 random moves to scramble the cube. 

![random]({{ site.baseurl }}/images/random_cube.png){: width="200"}

The "superflip" button performs a sequence of moves that puts the cube in the superflip configuration, where every edge is in the correct spot but flipped to the wrong orientation. The superflip is 20 moves away from being solved, which is the farthest any cube can be from being solved.

![superflip]({{ site.baseurl }}/images/superflip.png){: width="200"}

In the future I might add a "solve" button, which would require calculating a sequence of moves that solves the cube. 

# Cubelets

The 3x3x3 Rubik's cube can be thought of as a collection of 27 smaller "cublets". The innermost cubelet is never visible, so it doesn't have to be rendered.

I represented these small cubes with a `Cubelet` class. it stores the coordinates of the center of the cubelet and 3 rotation angles, one for each axis (x, y, z).
Each cubelet is initialized with the same color scheme as the solved cube, but many of the faces are not visible.
It's easiest to initialize every cubelet the same way.
(Also each cubelet stores its rotation radius, which depends on whether it's an edge, corner, or center piece. A center piece rotates in place.

Each of the 6 faces of the cubelet is a `Poly3DCollection` object (with a specified color).
The cubelets have a `update_position` function that uses trigonometry to calculate the locations of (all) the vertices from the current center position and rotation angles.
(rename that function).

Here is an example of the code for rotating the front face of the cubelet about the y axis by angle `th_y`. A cube can always (get to its current position by a rotation about only one axis), so only one rotation angle needs to be considered.
{% highlight Python %}
#front face
xs = [self.x+(np.sqrt(2)/2)*np.cos(5*np.pi/4 + self.th_y), self.x+(np.sqrt(2)/2)*np.cos(3*np.pi/4 + self.th_y), \
      self.x+(np.sqrt(2)/2)*np.cos(np.pi/4 + self.th_y), self.x+(np.sqrt(2)/2)*np.cos(-np.pi/4 + self.th_y)]
ys = [self.y-0.5, self.y-0.5, self.y-0.5, self.y-0.5]
zs = [self.z+(np.sqrt(2)/2)*np.sin(5*np.pi/4 + self.th_y), self.z+(np.sqrt(2)/2)*np.sin(3*np.pi/4 + self.th_y), \
      self.z+(np.sqrt(2)/2)*np.sin(np.pi/4 + self.th_y), self.z+(np.sqrt(2)/2)*np.sin(-np.pi/4 + self.th_y)]
verts1 = [list(zip(xs, ys, zs))]
{% endhighlight %}

# Animation

When a cube face (?) is supposed to rotate, here's how the animation works.
First, the nine cubelets belonging to that face are identified (one center, four corners, and four edges). (Always the same indices).
(There is a loop where) For each frame of the animation, (the function) calculates the position and rotation angle of each cubelet.
(Currently I am using 6 frames per animation).
For example:
{% highlight Python %}
d_theta = current_frame * np.pi / (2 * self.num_frames)
angle = np.arccos((cubelet.x0 - 1) / cubelet.rotation_radius) + d_theta
{% endhighlight %}
Then the `update_positions` function is called on all the cubelets to recalculate their vertices.
Finally, `plt.pause` is called which triggers the GUI to re-draw the figure, displaying the next frame of the animation.

Resetting:
At the end of the animation, once the face has turned a full 90 degrees, there is a problem. Let's say the front face just turned. All the cubelets that were part of the front face are still part of the front face... but three of the cubelets from the left face moved over to the right face, and so on. Now how does the function that rotates the left face know which cubelets need to rotate?
The cubelets are tracked by a (Python) list, so one option would be to shuffle the list. But I would still need to track the orientation of the cubelets. Instead, I decided to just reset all the cubelets to their original positions and update the colors at the same time so it looks like nothing happened.


[puzzle-code-repo]: https://github.com/bchellew15/rubiks_cube_simulator