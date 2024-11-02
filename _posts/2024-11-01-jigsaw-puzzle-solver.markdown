---
layout: post
title:  "Jigsaw Puzzle Solver"
date:   2024-11-01 06:22:45 -0700
categories: jekyll update
---

The goal of this project was to create a program that can solve jigsaw puzzles. The inputs are photos of the individual pieces, and the output is an image of the fully-assembled puzzle. [Here][puzzle-code-repo] is the code, written in C++ and using OpenCV.

I am not the first to attempt this. For example, Mark Rober and Stuff Made Here on YouTube have built jigsaw-puzzle-solving robots that required a program similar to this. My goal was to make this a learning experience for myself and to get practice with C++ and OpenCV.

By the way, I came across a similar project that also included the photo on the box as one of the inputs, but I wanted my program to solve the puzzle without using the box.

[comment]: # talk about wanting to be able to use a phone in non-perfect lighting conditions
[comment]: # (maybe: tried not to look at methods used by other projects bc wanted to figure out by myself.)
[comment]: # cut: I enjoy solving jigsaw puzzles and have wondered for a long time how hard it would be to get a computer to do it for me.
[comment]: # maybe: talk about assumptions about rectangular puzzle, standard shapes.

---

## How It Works

# Processing

The program begins by processing each puzzle piece image. (Images include a paper circle for scaling purposes.)

![original image]({{ site.baseurl }}/images/original_piece.jpeg){: width="200"}

Detect the background color and apply color-based thresholding to get a binary image.

[comment]: # explain how I determine the background color?

![binary image]({{ site.baseurl }}/images/bw_mask.png){: width="200"}

[comment]: # todo: choose a better example piece and crop properly
[comment]: # better image placement?

Use OpenCV's `findCountours` function to get the outline of the piece. (Identify which is the circle by comparing area and perimeter.)

![piece outline]({{ site.baseurl }}/images/outline.png){: width="200"}

[comment]: # todo: need thicker lines for bounding

Identify the "core" of the piece by drawing a bounding box and (iteratively) shrinking the bounding box (in each direction) until the intersection with the puzzle piece makes up more than half the size of the box.
(Note that this algorithm relies on multiple assumptions, including 1. that the puzzle piece is oriented correctly in the photo, and 2. that the tabs of the piece are not too wide or too tall.)

[comment]: # explain more. maybe more visual aids. say that I came up with the alg, and why is the alg like this.
[comment]: # talk about alternate alg to identify corners using slope? (this one was pretty simple and works well enough in my testing.)

![bounding box]({{ site.baseurl }}/images/bounding_box.png){: width="200"}
![piece core]({{ site.baseurl }}/images/core.png){: width="200"}

Divide the outline into four edges. Split the outline at the points that are closest to the corners of the piece core.

![separated edges]({{ site.baseurl }}/images/edges.png){: width="200"}

[comment]: # todo: check what other processing is done. e.g. identifying flat edges, creating raster images of the pieces.

# Match Function

The foundation of the assembly algorithm is the `match` function, which takes two puzzle piece edges and outputs a score indicating how well they fit together.
(edit) I decided to (represent) each piece edge as a binary image

![raster edge]({{ site.baseurl }}/images/raster_edge.png){: width="200"}

(todo: write about the overlap and stuff)

![piece match]({{ site.baseurl }}/images/edge_match.png){: width="200"}
![piece mismatch]({{ site.baseurl }}/images/edge_mismatch.png){: width="200"}

[comment]: # would be nice to add captions: match, mismatch

Here is the score calculation:
{% highlight c++ %}
bitwise_nor(edge1, edge2, nor_matrix);
bitwise_and(edge1, edge2, and_matrix);
return sum(nor_matrix)[0] + sum(and_matrix)[0];
{% endhighlight %}

The `and` penalizes overlap between the pieces, while the `nor` penalizes gaps between them. Note that a lower score is better, so that 0 is perfect match.
To get a final score, I rotate and translate (the second piece) (a few different value) and return the best score.
(something about figuring out the right relative orientation)

Speed is a critical because this match function is called many times. There are something like 4N^2 comparisons, where N is the number of pieces. And the `match` function is called several times for each edge comparison. This is why I chose to represent the piece edges as raster images and do bitwise operations, which are fast.

[comment]: # talk about how much time it takes?
[comment]: # I can't count on the corners being accurate
[comment]: # I tried Hausdorff distance, was very slow.

# Assembly

First a corner is identified and assumed to be the top left corner. After the puzzle is fully assembled, the user can rotate as needed.

![top left]({{ site.baseurl }}/images/topleft.png){: width="200"}

The top edge is assembled. Each edge piece is scored against the top left corner and the best match is chosen. This is repeated until the best match has a flat right edge, indicating that the top row is complete.

![top row 1]({{ site.baseurl }}/images/toprow1.png){: width="200"}
![top row complete]({{ site.baseurl }}/images/toprow2.png){: width="200"}

The left edge of the puzzle is assembled in the same way.

![left column 1]({{ site.baseurl }}/images/leftcol1.png){: width="200"}
![left column complete]({{ site.baseurl }}/images/leftcol2.png){: width="200"}

The rest of the puzzle is filled in. For each slot, the remaining pieces are scored in all allowed orientations, and the best match is placed there. The likelihood of identifying the correct piece is increased by scoring against the piece above the slot and the piece to the left of the slot and averaging the scores.

![filling in 1]({{ site.baseurl }}/images/fillin1.png){: width="200"}
![filling in 2]({{ site.baseurl }}/images/fillin2.png){: width="200"}
![completed puzzle]({{ site.baseurl }}/images/full_puzzle.png){: width="200"}

[comment]: # talk about how only allowed orientations are checked, to save time and prevent errors?
[comment]: # talk about fixing the rotation when assembling the edge?
[comment]: # maybe: talk about accuracy and avoiding wrong fits (e.g. assembling whole frame would be faster, but less likely to succeed bc less accurate)

# Display

Even after we know which pieces connect to which, it's not trivial to place each piece in the correct location in the completed puzzle image.



(at least, without big gaps and rotated pieces.)
(could just create a grid, but the pieces can be varying heights / widths. 
(todo: talk about calculating corrections...)
(and just calculating the position, I think it's based on the core midpoints. I can make some graphics for this.)
(maybe adding the numbers)
(creating masks and cutouts...)

Here is the completed puzzle!

![completed puzzle]({{ site.baseurl }}/images/full_puzzle.png)

[puzzle-code-repo]: https://github.com/bchellew15/puzzle_solver
