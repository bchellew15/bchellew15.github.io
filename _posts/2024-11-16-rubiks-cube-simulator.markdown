---
layout: post
title:  "Rubik's Cube Simulator (Python)"
date:   2024-11-16 06:22:45 -0700
categories: jekyll update
---

[The code for this project is [here][puzzle-code-repo], written in Python.]

I made a Rubik's cube simulator in Python. I used matplotlib to display and animate the cube because I'm already familiar with Matplotlib for plotting.

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

The "Superflip" button executes a sequence of moves that puts the cube in the superflip configuration, where every edge is in the correct spot but flipped to the wrong orientation. A cube in the superflip configuration is 20 moves away from being solved, which is the farthest any cube can be from being solved.

![superflip]({{ site.baseurl }}/images/superflip.png){: width="200"}

In the future I might add a "solve" button, which would require calculating a sequence of moves that solves the cube. 

# Cubelets

The 3x3x3 Rubik's cube can be thought of as a collection of 27 smaller "cublets." There are 12 edges, 8 corners, and 6 centers. The innermost cubelet is not visible.

I created a `Cubelet` class to represent these small cubes. It stores the coordinates of the cubelet's center and 3 rotation angles, one for each axis (x, y, z).
Each cubelet face is a `Poly3DCollection` object with 4 vertices and a specified color.
Each cubelet is initialized with the same color scheme as the solved cube.

The `update_position` function uses trigonometry to calculate the locations of the cubelet vertices from the center position and rotation angles. Here is the code for the front face of a cubelet centered on (`self.x`, `self.y`, `self.z`) and rotated by `self.th_y` about the y-axis:

{% highlight Python %}
#front face
xs = [self.x + (np.sqrt(2)/2) * np.cos(5*np.pi/4 + self.th_y), self.x + (np.sqrt(2)/2) * np.cos(3*np.pi/4 + self.th_y), \
      self.x + (np.sqrt(2)/2) * np.cos(np.pi/4 + self.th_y), self.x + (np.sqrt(2)/2) * np.cos(-np.pi/4 + self.th_y)]
ys = [self.y - 0.5, self.y - 0.5, self.y - 0.5, self.y - 0.5]
zs = [self.z + (np.sqrt(2)/2) * np.sin(5*np.pi/4 + self.th_y), self.z + (np.sqrt(2)/2) * np.sin(3*np.pi/4 + self.th_y), \
      self.z + (np.sqrt(2)/2) * np.sin(np.pi/4 + self.th_y), self.z + (np.sqrt(2)/2)*np.sin(-np.pi/4 + self.th_y)]
verts = [list(zip(xs, ys, zs))]
{% endhighlight %}

# Animation

When a cube face turns there is a 6-frame animation that plays. 

First, the nine cubelets belonging to that face are identified - a center, four corners, and four edges.

For each frame of the animation, (the function) calculates the position and rotation angle of each cubelet.
Some pieces move more than others. Corners have a larger rotation radius than edges, while center pieces rotate in place.
(Each cubelet stores its rotation radius)

For example:
{% highlight Python %}
d_theta = current_frame * np.pi / (2 * self.num_frames)
angle = np.arccos((cubelet.x0 - 1) / cubelet.rotation_radius) + d_theta
{% endhighlight %}

Then the `update_positions` function is called on all the cubelets to recalculate their vertices.

A cube can always (get to its current position by a rotation about only one axis), so only one rotation angle needs to be considered.

Finally, `plt.pause` is called which triggers the GUI to re-draw the figure, displaying the next frame of the animation.

Resetting:
At the end of the animation, once the face has turned a full 90 degrees, there is a problem. Let's say the front face just turned. All the cubelets that were part of the front face are still part of the front face... but three of the cubelets from the left face moved over to the right face, and so on. Now how does the function that rotates the left face know which cubelets need to rotate?
The cubelets are tracked by a (Python) list, so one option would be to shuffle the list. But I would still need to track the orientation of the cubelets. Instead, I decided to just reset all the cubelets to their original positions and update the colors at the same time so it looks like nothing happened.


[puzzle-code-repo]: https://github.com/bchellew15/rubiks_cube_simulator