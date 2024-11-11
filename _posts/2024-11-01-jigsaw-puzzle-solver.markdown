---
layout: post
title:  "Jigsaw Puzzle Solver (C++, OpenCV)"
date:   2024-11-01 06:22:45 -0700
categories: jekyll update
---

[The code for this project is [here][puzzle-code-repo], written in C++ and using OpenCV.]

I wrote a program that solves jigsaw puzzles. The inputs are photos of the individual pieces, and the output is an image of the fully-assembled puzzle. I wanted it to work for a 100 piece puzzle off the shelf, with pictures taken by a smartphone in sub-optimal lighting conditions.

I made some assumptions about the puzzle. The border should be rectangular and the pieces should have standard shapes and be arranged in a grid-cut pattern. 

I am not the first to attempt this. For example, Mark Rober and Stuff Made Here on YouTube have built jigsaw-puzzle-solving robots that required a program similar to this. My goal was to practice C++ and OpenCV and make this a learning experience for myself.

---

## How It Works

# Processing

Read each image into an OpenCV `Mat` object that is a property of a `PuzzlePiece` object. Images include a paper circle for scaling purposes.

![original image]({{ site.baseurl }}/images/original_piece.jpeg){: width="200"}

Apply color-based thresholding to get a binary image. This is done by converting to HSV color space with `cvtColor`, then sampling the corners of the image to detect the hue (H) and saturation (S) of the background. The hue and saturation ranges are increased slightly to account for uneven lighting. Shadows cast by the puzzle piece are darker than the background, so the values in the V channel are smaller, but they have mostly the same hue and saturation so they are still categorized as background.

![binary image]({{ site.baseurl }}/images/bw_mask.png){: width="200"}

Use OpenCV's `findCountours` function to get the outline of the piece. Identify which contour is the circle by comparing area and perimeter, then scale the image based on the size of the circle.

![piece outline]({{ site.baseurl }}/images/outline.png){: width="200"}

Identify what I will call the "core rectangle" of the piece by drawing a bounding box around the puzzle piece outline and then shrinking the box from each side until the intersection between the box and the puzzle piece outline is more than half the width of the box.

This algorithm assumes that the puzzle piece is oriented correctly in the photo and that the tabs of the piece are not too wide or too tall.
An algorithm that starts by locating the corners of the piece would be better in several ways. Photos could be taken with pieces rotated in any orientation, and pieces could be matched quickly and accurately by comparing the distance between corners and lining up the corners without trial and error.
I have written a corner detection algorithm based on changes in the slope of a fit line, but it will take time to make it robust.
In the meantime, this algorithm was easy to implement and has worked well enough.

![bounding box]({{ site.baseurl }}/images/bounding_box.png){: width="200"}
![piece core]({{ site.baseurl }}/images/core.png){: width="200"}

Divide the outline into four sections, which I will call "edges." Split the outline at the points that are closest to the corners of the core rectangle. An `EdgeOfPiece` object is created for each edge.

![separated edges]({{ site.baseurl }}/images/edges.png){: width="200"}

Finally, each edge is categorized as a tab, a blank, or a flat edge.

# Match Function

The foundation of the assembly algorithm is the `matchEdges` function, which takes two puzzle piece edges and outputs a score indicating how well they fit together. 

There is some trial and error involved to make sure the pieces are properly aligned with each other. A `score` function is called multiple times with one of the pieces shifted different amounts in the x- and y-directions and rotated by different angles, then the best score is returned. If I implement a corner detection algorithm, as described above, trial and error will not be necessary.

Speed is critical because the `score` function is called so many times. There are ~N^2 edge comparisons, where N is the number of pieces, and the `score` function is called potentially hundreds of times for each edge comparison. The puzzle assembly currently runs in 1 second for a 16 piece puzzle, which corresponds to 40 seconds for a 100 piece puzzle or over an hour for a 1000 piece puzzle.

I initially tried to measure shape similarity using the Hausdorff distance. 
The piece outline is stored as a set of points, and this involved iterating through each pair of points.
The method was only somewhat effective, and it was slow -- O(N^2), where N is a large number of points.

I decided to represent each edge as a binary raster image (using the `drawContour` function) and compare them with bitwise operations, which are fast. I also decreased the resolution of the images to make it faster.

![raster edge]({{ site.baseurl }}/images/raster_edge.png){: height="100"}

Here are examples of edge comparisons, with one edge colored red and the other blue. Overlap between the pieces is purple and gaps between them are black.
The left image shows a correct match while the right image shows an incorrect match.

![piece match]({{ site.baseurl }}/images/edge_match.png){: height="130"}
![piece mismatch]({{ site.baseurl }}/images/edge_mismatch.png){: height="130"}

Here is the score calculation:
{% highlight c++ %}
bitwise_nor(edge1, edge2, nor_matrix);
bitwise_and(edge1, edge2, and_matrix);
return sum(nor_matrix)[0] + sum(and_matrix)[0];
{% endhighlight %}

The idea is to penalize overlap between pieces and penalize gaps between them. `and` is used to count the number of overlap pixels, while `nor` is used to count the number of gap pixels. Note that a lower score is better, and a perfect match with 0 overlap pixels and 0 gap pixels would get a score of 0.

# Assembly

To reduce errors and save time, pieces are only scored against each other in orientations where the connection is allowed. For example, a tab can be scored against a blank, but it cannot be scored against a tab or a flat edge. A connection is also not allowed if a flat edge would be adjacent to a tab or blank. On average, a piece has one allowed orientation in a given spot.

As pieces are connected to the puzzle, pointers are added to a 2D `vector` of `PuzzlePiece` objects.

First, a corner is identified and assumed to be the top left corner. The user can rotate as needed after the puzzle is fully assembled.

![top left]({{ site.baseurl }}/images/topleft.png){: height="150"}

The top edge is assembled. The `match` function is called, which scores each allowed piece against the top left corner and returns the best match. Then `match` is called on the best match, and so on until the best match has a flat right edge, indicating that the top row is complete.

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

My next thought was to place pieces based on the core rectangles found in the processing step, lining up the centers of the core edges. This is an improvement, but there are still corrections that need to be made. These corrections can be calculated from the shift and rotation that resulted in the best score between the pieces. The rotation correction is added to the actual rotation applied to the neighboring pieces.

When placing a piece in a slot, its position could be determined based on the piece above the slot or the piece to the left of the slot. In theory both of these options would result in the same placement, but in practice the neighboring pieces are not perfectly placed and the corrections are not exact. I chose to calculate a position and rotation angle based on each of the neighboring pieces and take the average.

The piece outline contour is used to create a mask so that only the relevant pixels are copied using the `copyTo` function.

Finally, any remaining gaps are filled using a median filter. Here is the completed puzzle:

![completed puzzle]({{ site.baseurl }}/images/fill_gaps.png)

There is an option to superimpose the corresponding image number on each piece. If the pieces are numbered in advance - for example, by writing numbers on the back with a sharpie - this allows the user to easily assemble the physical puzzle.

![fill gaps]({{ site.baseurl }}/images/numbers.png){: height="150"}

[puzzle-code-repo]: https://github.com/bchellew15/puzzle_solver
