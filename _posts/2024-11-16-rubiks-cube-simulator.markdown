---
layout: post
title:  "Rubik's Cube Simulator (Python)"
date:   2024-11-16 06:22:45 -0700
categories: jekyll update
---

[The code for this project is [here][puzzle-code-repo], written in Python.]

I made a Rubik's cube simulator with Python, using Matplotlib to display and animate the cube.

Here is a demonstration video:
<video width="320" height="240" controls loop="" muted="" autoplay="">
   <source src="https://github.com/bchellew15/bchellew15.github.io/raw/refs/heads/main/images/rubiks_screen_record_480.mp4">
</video>

---
<br>

# User Interface

I drew the cube with Matplotlib's 3D plotting module, `mpl_toolkits.mplot3d`.
The Matplotlib GUI allows the user to click and drag to view the cube from different angles, even while animations are running.

For buttons I used `matplotlib.widgets.Button`. The lower buttons turn the cube faces. They are labeled using standard notation for Rubik's cube face rotations. F is the front face, R is right, U is up, B is back, L is left, and D is down. The default turn direction is clockwise, and prime (') represents a counter-clockwise turn, so clicking the R' button turns the right face counter-clockwise.

When a button is clicked, the face rotation is added to a queue and executed once any pending face rotations are completed.

The upper buttons are "Reset," "Randomize," and "Superflip." The "Reset" button resets the colors. The "Randomize" button executes 20 random moves to scramble the cube. 

![random]({{ site.baseurl }}/images/random_cube.png){: width="200"}

The "Superflip" button executes a sequence of moves that puts the cube in the superflip configuration, where every edge is in the correct spot but flipped to the wrong orientation. A cube in the superflip configuration is 20 moves away from being solved, which is the furthest any cube can be from being solved.

![superflip]({{ site.baseurl }}/images/superflip.png){: width="200"}

In the future I might add a "solve" button, which could be done by calculating a sequence of moves that solves the cube, or by keeping track of all the turns so far and reversing them. 

# Cubelets

The 3x3x3 Rubik's cube can be thought of as a collection of 27 smaller "cublets." There are 12 edges, 8 corners, and 6 centers. The innermost cubelet is not visible.

I created a `Cubelet` class to represent these small cubes. It stores the coordinates of the cubelet's center and 3 rotation angles, one for each axis (x, y, z).
Each cubelet face is a `Poly3DCollection` object with 4 vertices and a specified color.
Each cubelet is initialized with the same color scheme as the solved cube.

The `update_position` function uses trigonometry to calculate the locations of the cubelet vertices from the center position and rotation angles. Here is the code for the front face of a cubelet centered on (`self.x`, `self.y`, `self.z`) and rotated (in place) by `self.th_y` about the y-axis:

{% highlight Python %}
#front face
xs = [self.x + (np.sqrt(2)/2) * np.cos(5*np.pi/4 + self.th_y), self.x + (np.sqrt(2)/2) * np.cos(3*np.pi/4 + self.th_y), \
      self.x + (np.sqrt(2)/2) * np.cos(np.pi/4 + self.th_y), self.x + (np.sqrt(2)/2) * np.cos(-np.pi/4 + self.th_y)]
ys = [self.y - 0.5, self.y - 0.5, self.y - 0.5, self.y - 0.5]
zs = [self.z + (np.sqrt(2)/2) * np.sin(5*np.pi/4 + self.th_y), self.z + (np.sqrt(2)/2) * np.sin(3*np.pi/4 + self.th_y), \
      self.z + (np.sqrt(2)/2) * np.sin(np.pi/4 + self.th_y), self.z + (np.sqrt(2)/2)*np.sin(-np.pi/4 + self.th_y)]
verts = [list(zip(xs, ys, zs))]
{% endhighlight %}

Only one rotation angle needs to be considered at a time because face rotations always start with every cubelet aligned with the coordinate axes.

# Animation

When a cube face turns, a 6-frame animation plays.

First, the nine cubelets belonging to that face are identified -- a center, four corners, and four edges.

For each frame of the animation, a rotation angle is calculated based on the current frame number divided by the total number of frames.
Then the position of each cubelet's center is calculated from this rotation angle and the cubelet's rotation radius, which is its distance from the center of the face. Corners have a larger rotation radius than edges, while center pieces have a rotation radius of 0 because they rotate in place.

Here is the code for a frame when the front face is rotating:
{% highlight Python %}
d_theta = current_frame * np.pi / (2 * self.num_frames)
cubelet.th_y = d_theta

# "angle" is the angular position of the cubelet with the origin at the center of the face and polar axis pointing to the right
angle = np.arccos((cubelet.x0 - 1) / cubelet.rotation_radius) + d_theta
cubelet.x = 1 + cubelet.rotation_radius * np.cos(angle)
cubelet.z = 1 + cubelet.rotation_radius * np.sin(angle)
{% endhighlight %}

Then the `update_position` function is called on each cubelet, as described earlier, to calculate its vertices.

Finally `plt.pause` is called, which triggers the GUI to re-draw the figure, displaying the next frame of the animation.

#### Resetting

After a face rotates, cubelets have moved from their starting positions, they are in different orientations, and they are part of different faces of the cube. The next animation needs to know which cubelets are in which faces and what their starting positions / orientations are.

I could have updated all these things after each animation, but it would have required slightly different code for each of the 12 face rotations, so I looked for an easier way. I also would have had to deal with error accumulation.

I ended up using a trick. After a face is done turning, the cubelets reset to their starting positions, but at the same time the colors update so it still looks like the face has turned.
This way the animation for each face always moves the same `Cubelet` objects from the same starting positions / orientations.

[puzzle-code-repo]: https://github.com/bchellew15/rubiks_cube_simulator