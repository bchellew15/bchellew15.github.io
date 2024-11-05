---
layout: post
title:  "Jigsaw Puzzle Solver"
date:   2024-11-01 06:22:45 -0700
categories: jekyll update
---

The goal of this project was to create a program that can solve jigsaw puzzles. The inputs are photos of the individual pieces, and the output is an image of the fully-assembled puzzle. The code is [here][puzzle-code-repo], written in C++ and using OpenCV.

I made some assumptions about the puzzle. It should have a rectangular border and the pieces should have standard shapes, roughly rectangular with tabs and blanks.

I am not the first to attempt this. For example, Mark Rober and Stuff Made Here on YouTube have built jigsaw-puzzle-solving robots that required a program similar to this. My goal was to make this a learning experience for myself and to get practice with C++ and OpenCV.

---

## How It Works

# Processing

The program begins by processing each puzzle piece image. Images include a paper circle for scaling purposes.

![original image]({{ site.baseurl }}/images/original_piece.jpeg){: width="200"}

Detect the background color and apply color-based thresholding to get a binary image.

![binary image]({{ site.baseurl }}/images/bw_mask.png){: width="200"}

Use OpenCV's `findCountours` function to get the outline of the piece. Identify which is the circle by comparing area and perimeter, then scale the image based on the size of the circle.

![piece outline]({{ site.baseurl }}/images/outline.png){: width="200"}

Identify the "core rectangle" of the piece by drawing a bounding box and shrinking it in each dimension until the intersection with the puzzle piece makes up more than half the size of the box.
(Note that this algorithm relies on multiple assumptions, including 1. that the puzzle piece is oriented correctly in the photo, and 2. that the tabs of the piece are not too wide or too tall.)
An algorithm that reliably locates the corners would be better in several ways. (can photograph in any orientation, can align without trial and error -> speedup)
I have tried some corner detection algorithms, but it will take time to make them more robust.
This algorithm was simple to implement and has worked well enough for now.

![bounding box]({{ site.baseurl }}/images/bounding_box.png){: width="200"}
![piece core]({{ site.baseurl }}/images/core.png){: width="200"}

Divide the outline into four edges. Split the outline at the points that are closest to the corners of the piece core.

![separated edges]({{ site.baseurl }}/images/edges.png){: width="200"}

Finally, each edge of the piece is labeled as a tab, a blank, or a flat edge.

# Match Function

The foundation of the assembly algorithm is the `match` function, which takes two puzzle piece edges and outputs a score indicating how well they fit together. 

(edit) To get a final score, I rotate and translate (the second piece) (a few different value) and return the best score.
(edit) (there is score function, but calculate score for various translations / rotations)
If I could reliably detect the corners, as described above, the pieces could be quickly aligned rather than using trial and error.

I initially tried some algorithms to calculate shape similarity. For example, I tried calculating the Hausdorff distance. 
The piece outline is stored by OpenCV as a countour, which is a set of points, which involves iterating through each pair of points.
This method was only somewhat effective, and it was slow.

Speed is a critical because this match function is called many times. There are ~N^2/2 comparisons, where N is the number of pieces, and the `score` function is called several times for each edge comparison. 

I decided to represent each piece edge as a binary raster image and compare them with bitwise operations, which are fast. I also decreased the resolution of the images because the bitwise operations are faster with fewer pixels.

![raster edge]({{ site.baseurl }}/images/raster_edge.png){: height="100"}

Here are examples of piece comparisons, with one edge colored red and the other colored blue. Overlap between the pieces is purple and gaps between them are black.
On the left is a correct match and on the right is an incorrect match.

![piece match]({{ site.baseurl }}/images/edge_match.png){: height="130"}
![piece mismatch]({{ site.baseurl }}/images/edge_mismatch.png){: height="130"}

Here is the score calculation:
{% highlight c++ %}
bitwise_nor(edge1, edge2, nor_matrix);
bitwise_and(edge1, edge2, and_matrix);
return sum(nor_matrix)[0] + sum(and_matrix)[0];
{% endhighlight %}

The idea is to penalize overlap between and pieces and penalize gaps between them. `and` is used to count the number of overlap pixels, while `nor` is used to count the number of gap pixels. Note that a lower score is better, and a perfect match with 0 overlap pixels and 0 gap pixels would get a score of 0.

# Assembly

To reduce errors and save time, two pieces are only scored against each other in orientations where the connection is "allowed". This means, for example, a tab is only allowed to be compared to a blank, not a tab or a flat edge. A connection is also not allowed if it means a flat edge will be adjacent to a tab or blank. On average, a piece will have one allowed orientation in a given spot.

First, a corner is identified and assumed to be the top left corner. The user can rotate as needed after the puzzle is fully assembled.

![top left]({{ site.baseurl }}/images/topleft.png){: height="150"}

The top edge is assembled. Each edge piece is scored against the top left corner and the best match is chosen. This is repeated until the best match has a flat right edge, indicating that the top row is complete.

![top row 1]({{ site.baseurl }}/images/toprow1.png){: height="150"}
![top row complete]({{ site.baseurl }}/images/toprow2.png){: height="150"}

The left edge of the puzzle is assembled in the same way.

![left column 1]({{ site.baseurl }}/images/leftcol1.png){: height="150"}
![left column complete]({{ site.baseurl }}/images/leftcol2.png){: height="150"}

The rest of the puzzle is filled in. For each slot, the remaining pieces are scored in all allowed orientations, and the best match is placed there. The likelihood of identifying the correct piece is increased by scoring against the piece above the slot and the piece to the left of the slot and averaging the scores.

![filling in 1]({{ site.baseurl }}/images/fillin1.png){: height="150"}
![filling in 2]({{ site.baseurl }}/images/fillin2.png){: height="150"}
![completed puzzle]({{ site.baseurl }}/images/full_puzzle.png){: height="150"}

# Display

Even after we know which pieces connect to which, it's not trivial to place each piece in the correct location in the completed puzzle image. My first approach was to place the pieces on an evenly spaced grid based on the size of the first piece. But the pieces can have different widths and heights, so this leaves many gaps and overlaps. The pieces might also need to be rotated a little, despite efforts to align them when taking the pictures.

My next thought was to place pieces based on the core rectangles found in the processing step, lining up the centers of the core edges. This is an improvement, but there are still corrections that need to be made. These corrections can be calculated from the shift and rotation that resulted in the best score between the pieces.

When placing a piece in a slot, its position could be determined based on the piece above the slot or the piece to the left of the slot. In theory both of these options would result in the same placement, but in practice the neighboring pieces are not perfectly placed and the corrections are not exact. I chose to calculate a position and rotation angle based on each of the neighboring pieces and take the average.

(cut?) The piece outline is used to create a mask so that only the relevant pixels are copied.

Finally, any remaining gaps are filled using a median filter. Here is the completed puzzle:

![completed puzzle]({{ site.baseurl }}/images/fill_gaps.png)

There is an option to superimpose the corresponding image number on each piece. If the pieces are numbered in advance - for example, by writing numbers on the back with a sharpie - this allows the user to easily assemble the physical puzzle.

![fill gaps]({{ site.baseurl }}/images/numbers.png){: height="150"}

[puzzle-code-repo]: https://github.com/bchellew15/puzzle_solver
