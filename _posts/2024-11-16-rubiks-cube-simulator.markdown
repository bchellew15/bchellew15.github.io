---
layout: post
title:  "Rubik's Cube Simulator (Python)"
date:   2024-11-16 06:22:45 -0700
categories: jekyll update
---

[The code for this project is [here][puzzle-code-repo], written in Python.]

I made a Rubik's cube simulator in Python.
I used matplotlib to display the cube and animate face rotations, because I am already familiar with matplotlib from making plots (for physics assignments, research)
I enjoy solving rubiks cubes.

---

How it works (?):

The 3x3x3 Rubik's cube can be thought of as a collection of 27 smaller "cublets" (?). The center / innermost cubelet is never visible, so I don't have to render / consider it. And if you take apart a rubik's cube, you will find the internal mechanisms (core) in that spot.
(In real cubes the parts are not actually cube shaped).

So I began with a Cubelet class for these small cubes. it stores the coordinates of the center of the cube and 3 rotation angles, one for each axis (x, y, z)
Each cubelet has (each of the six colors) (not explained). Even though only some of them are visible, it's easier to just initialize every cubelet the same way.
(Also each cubelet stores its rotation radius, which depends on whether it's an edge, corner, or center piece. A center piece rotates in place.

Each of the 6 faces of the cubelet is a `Poly3DCollection` object (with a specified color).
The cubelets have a `update_position` function that uses trigonometry to calculate the locations of (all) the vertices from the current center position and rotation angles.
(rename that function).

Here is an example of the code for rotating the front face of the cubelet about the y axis by angle `thy`. A cube can always (get to its current position by a rotation about only one axis), so only one rotation angle needs to be considered.
{% highlight Python %}
#front face
            xs = [self.x+(np.sqrt(2)/2)*np.cos(5*np.pi/4 + self.thy), self.x+(np.sqrt(2)/2)*np.cos(3*np.pi/4 + self.thy), \
                  self.x+(np.sqrt(2)/2)*np.cos(np.pi/4 + self.thy), self.x+(np.sqrt(2)/2)*np.cos(-np.pi/4 + self.thy)]
            ys = [self.y-0.5, self.y-0.5, self.y-0.5, self.y-0.5]
            zs = [self.z+(np.sqrt(2)/2)*np.sin(5*np.pi/4 + self.thy), self.z+(np.sqrt(2)/2)*np.sin(3*np.pi/4 + self.thy), \
                  self.z+(np.sqrt(2)/2)*np.sin(np.pi/4 + self.thy), self.z+(np.sqrt(2)/2)*np.sin(-np.pi/4 + self.thy)]
            verts1 = [list(zip(xs, ys, zs))]
{% endhighlight %}

# GUI

Here is a screenshot of the (simulator).
(todo: add screenshot)

I used the matplotlib GUI, calling `plt.figure()` and plt.show().
A benefit is that I can click and drag to view the cube from different angles, even while animations are running, with no extra work on my part.
There are buttons to move each of the faces, labeled using standard Rubik's cube notation. F is the front face, R is right, U is upper, B is back, L is left, D is down. The (default) turn direction is clockwise. Prime (') represents a counter-clockwise turn, so clicking the R' button turns the right face counter-clockwise.
(todo: add demonstration video / gif)

(edit) All (these functions) assume that other face rotations are complete before they start. When a button is clicked, the (corresponding function) is added to a queue, so that any pending face rotations can be completed.

I added a few more buttons to perform high-level functions. The reset button is self-explanatory and just resets the colors. The "randomize" button executes 20 random moves to scramble the cube. The "superflip" button performs a sequence of moves that puts the cube in the "superflip" position, where every edge is in the correct spot but flipped to the wrong orientation. It is also one of the configurations where the optimal solution takes the maximum 20 moves (clarify).
(todo: add superflip image)
Maybe in the future I'll add a "solve" button, which would require me to calculate a sequence of moves that solves the cube. 

# Animation

When a cube face (?) is supposed to rotate, here's how it works.
First, the nine cubelets belonging to that face are identified (one center, four corners, and four edges). (Always the same indices).
(There is a loop where) For each frame of the animation, (the function) calculates the position and rotation angle of each cubelet.
(Currently I am using 6 frames per animation).
For example:
{% highlight Python %}
d_th = i*np.pi/(2*self.num_frames)
angle = np.arccos((c.x0-1)/c.r) + d_th
{% endhighlight %}
Then the `update_positions` function is called on all the cubelets to recalculate their vertices.
Finally, `plt.pause` is called which triggers the GUI to re-draw the figure, effectively displaying the next frame of the animation.

Resetting:
At the end of the animation, once the face has turned a full 90 degrees, there is a problem. Let's say the front face just turned. All the cubelets that were part of the front face are still part of the front face... but three of the cubelets from the left face moved over to the right face, and so on. Now how does the function that rotates the left face know which cubelets need to rotate?
The cubelets are tracked by a (Python) list, so one option would be to shuffle the list. Instead, for no good reason, I decided to reset all the cubelets to their original position and just update the colors so it still looks like the rotation happened.


[puzzle-code-repo]: https://github.com/bchellew15/rubiks_cube_simulator