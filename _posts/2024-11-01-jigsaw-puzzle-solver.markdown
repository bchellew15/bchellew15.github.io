---
layout: post
title:  "Jigsaw Puzzle Solver"
date:   2024-11-01 06:22:45 -0700
categories: jekyll update
---

The goal of this project was to create a program that can solve a jigsaw puzzle. The inputs are photos of the individual pieces, and the output is an image of the fully-assembled puzzle. The code is [here][puzzle-code-repo].

I am not the first to attempt this. For example, Mark Rober and Stuff Made Here on YouTube have built jigsaw-puzzle-solving robots that required a program similar to this. My goal was for this to be a learning experience for myself and to get practice with C++ and OpenCV.

By the way, I came across a similar project that also included the photo on the box as one of the inputs, but I wanted my program to solve the puzzle without using the box.

[comment]: # talk about wanting to be able to use a phone in non-perfect lighting conditions

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

![binary image]({{ site.baseurl }}/images/outline.png){: width="200"}

[comment]: # todo: need thicker lines for bounding

Identify the "core" of the piece by drawing a bounding box and (iteratively) shrinking the bounding box (in each direction) until the intersection with the puzzle piece makes up more than half the size of the box.
(Note that this algorithm relies on multiple assumptions, including 1. that the puzzle piece is oriented correctly in the photo, and 2. that the tabs of the piece are not too wide or too tall.)

[comment]: # explain more. maybe more visual aids. say that I came up with the alg, and why is the alg like this.
[comment]: # talk about alternate alg to identify corners using slope? (this one was pretty simple and works well enough in my testing.)

![binary image]({{ site.baseurl }}/images/bounding_box.png){: width="200"}
![binary image]({{ site.baseurl }}/images/core.png){: width="200"}

Divide the outline into four edges. Split at the points on the outline that are closest to the corners of the piece "core".

![binary image]({{ site.baseurl }}/images/edges.png){: width="200"}

[comment]; # todo: check what other processing is done. e.g. identifying flat edges, creating raster images of the pieces.

# Assembly

(placeholder)

# Display

(placeholder)

---

(maybe: tried not to look at methods used by other projects bc wanted to figure out by myself.)

cut: I enjoy solving jigsaw puzzles and have wondered for a long time how hard it would be to get a computer to do it for me.

`backticks example`

{% highlight python %}
# prints "hello world"
def hello:
  print("hello world")
{% endhighlight %}

[Jekyll docs][jekyll-docs]

[puzzle-code-repo]: https://github.com/bchellew15/puzzle_solver
[google]: google.com

[jekyll-docs]: https://jekyllrb.com/docs/home
