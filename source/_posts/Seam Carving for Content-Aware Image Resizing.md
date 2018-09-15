---
title: Seam Carving for Content-Aware Image Resizing
tags:
  - Image resizing
  - Image retargeting
  - Image seams
categories:
  - Computer Vision
date: 2018-08-13 15:26:29
---
論文筆記(待續)

# Introduction
Standard image scaling is not sufficient since it is oblivious to the
image content and typically can be applied only uniformly. Cropping
is limited since it can only remove pixels from the image periphery.
More effective resizing can only be achieved by considering
the image content and not only geometric constraints

Seam carving uses an energy function defining the importance of pixels
A seam is a connected path of low energy pixels crossing the image from top to
bottom, or from left to right By successively removing or inserting
seams we can reduce, as well as enlarge, the size of an image
in both directions. For image reduction, seam selection
ensures that while preserving the image structure, we remove
more of the low energy pixels and fewer of the high energy ones.
For image enlarging, the order of seam insertion ensures a balance
between the original image content and the artificially inserted pixels.
These operators produce, in effect, a content-aware resizing of images.

# The Operator
Our approach to content-aware resizing is to remove pixels in a judicious
manner. Therefor, the question is how to chose the pixels to
be removed? Intuitively, our goal is to remove unnoticeable pixels
that blend with their surroundings. This leads to the following simple
*energy function* \\(e(I)=|\frac{\partial}{\partial x}I|+|\frac{\partial}{\partial y}I|\\)

Given an energy function, assume we need to reduce the image
width. One can think of several strategies to achieve this. For instance,
an optimal strategy to preserve energy would be to remove the pixels with lowest energy
in ascending order. This destroys the rectangular shape of the
image, because we may remove a different number of pixels from
each row . If we want to prevent the image from
breaking we can remove an equal number of low energy pixels from
every row. This preserves the rectangular shape of the image but destroys
the image content by creating a zigzag effect .
To preserve both the shape and the visual coherence of the image
we can use Auto-cropping. That is, look for a sub-window, the size
of the target image, that contains the highest energy
Another possible strategy somewhat between removing pixels and
cropping is to remove whole columns with the lowest energy. Still,
artifacts might appear in the resulting image . Therefore,
we need a resizing operator that will be less restrictive than
cropping or column removal, but can preserve the image content
better than single pixel removals. This leads to our strategy of seam
carving  and the definition of internal seams.

Computing the seam consists of finding the path of minimum energy cost from
one end of the image to another. This can be done via Dijkstra's algorithm,
dynamic programming, greedy algorithm or graph cuts among others

## Image Energy Functions
We have examined several possible image importance measures
found in literature as the energy function to guide seam carving.
We have tested both L1 and L2-norm of the gradient, saliency
measure ,and Harris-corners measure. We also used eye gaze 
measurement and the output of face detectors.

$$ e_{HoG}(I)=\frac{|\frac{\partial}{\partial x}I|+|\frac{\partial}{\partial y}I|}{max(HoG(I(x,y)))}$$
where \\(HoG(I(x, y))\\) is taken to be a histogram of oriented gradients
at every pixel

# Discrete Image Resizing
## Aspect Ratio Change

Assume we want to change the aspect ratio of a given image **I** from
\\(n \times m\\) to \\(n \times {m}'\\) where \\(m-{m}'=c\\)
This can be achieved simply by successively removing c vertical seams from I.
Contrary to simple scaling, this operation will not alter important parts of the image (as
defined by the energy function), and in effect creates a non-uniform, content-aware resizing of the image

The same aspect ratio correction, from \\(n \times m\\) to \\(n \times {m}'\\)
can also be achieved by increasing the number of rows by a factor of \\(m/{m}'\\)
The added value of such an approach is that it does not remove any information from the image

## Retargeting with Optimal Seams-Order
Image retargeting generalizes aspect ratio change from one dimension
to two dimensions such that an image **I** of size \\(n \times m\\) will be
retargeted to size \\({n}' \times {m}'\\) and, for the time being, we assume that
\\({m}' < m\\) and \\({n}' < n\\)
This begs the question of what is the correct order of seam carving?
Remove vertical seams first? Horizontal seams first? Or alternate between the two?
We define the search for the optimal order as an optimization of the following objective function

## Seam Carving in the gradient domain
There are times when removing multiple seams from an image still
creates noticeable visual artifacts in the resized image. To overcome
this we can combine seam carving with Poisson reconstruction.
Specifically, we compute the energy function image as before,
but instead of removing the seams from the original image we
work in the gradient domain and remove the seams from the x and y
derivatives of the original image. At the end of this process we use
a poisson solver to reconstruct back the image. 

## Reference
[Seam Carving for Content-Aware Image Resizing](https://perso.crans.org/frenoy/matlab2012/seamcarving.pdf)