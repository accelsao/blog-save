---
title: Snakes Active Contour Models 
tags:
- Active contour model
categories:
- Computer Vision
date: 2018-08-12 13:03:50
---
Paper Notes (To be continued)

# Introduction
Active contour model, also called snakes, is a framework in computer vision introduced by Michael Kass, Andrew Witkin and Demetri Terzopoulos for delineating an object outline from a possibly noisy 2D image
應用於object tracking, shape recognition, segmentation, edge detection and stereo matching.

# Basic snake model
A snake is an energy minimizing, deformable spline influenced by constraint and image forces that pull it towards object contours and internal forces that resist deformation


Our basic snake model is a *controlled continuity* spline under the influence of image forces and external constraint forces.
Let position of snake be $$v(s)=(x(s),y(s))$$
then energy function is
\begin{split}
E^*_{snake}&=\int_{0}^{1}E_{snake}(v(s))ds \\
&=\int_{0}^{1}E_{int}(v(s))+E_{image}(v(s))+E_{con}(v(s)) ds
\end{split}
where
\\(E_{int}\\) internal energy
\\(E_{image}\\) image forces
\\(E_{con}\\) external constraint forces

## Internal Energy

The internal energy of the snake is composed of the continuity of the contour \\(E_{cont}\\) and the smoothness of the contour \\(E_{curv}\\)
$$E_{internal}=E_{cont}+E_{curv}$$

$$E_{{internal}}={\frac  {1}{2}}(\alpha \,\!(s)\left|{\mathbf  {v}}_{s}(s)\right\vert ^{2})+{\frac  {1}{2}}(\beta \,\!(s)\left|{\mathbf  {v}}_{{ss}}(s)\right\vert ^{2})={\frac  {1}{2}}{\bigg (}\alpha \,\!(s)\left\|{\frac  {d{\bar  v}}{ds}}(s)\right\Vert ^{2}+\beta \,\!(s)\left\|{\frac  {d^{2}{\bar  v}}{ds^{2}}}(s)\right\Vert ^{2}{\bigg )}$$

In practice, a large weight \\(\alpha (s)\\) for the continuity term penalizes changes in distances between points in the contour.
A large weight \\(\beta(s)\\) for the smoothness term penalizes oscillations in the contour and will cause the contour to act as a thin plate.



# Image Forces
Total image energy can be expressed as a weighted combination of the 3 energy functionals
$$E_{image}=w_{line}\cdot E_{line}+w_{edge}\cdot E_{edge}+w_{term}\cdot E_{term}$$

## Line Functional
假設 \\(E_{line} = I(X,Y)\\) 則snake會隨著\\(w_{line}\\)的sign被吸引向light lines 或 dark lines 
## Edge Functional 
Set \\(E_{edge} =-\left| \nabla I(x,y) \right|^2\\) 則snake被large image gradients 吸引
## Scale Space 
當部分snake找到 low-energy image feature,the spline term  will pull neighboring parts of the snake toward a possible continuation of the feature
This effectively places a large energy well around a good local minimum.
A similar effect can be achieved by spatially smoothing the edge or line-energy functional.
One can allow the snake to come to equilibrium on a very blurry energy functional and then slowly reduce the blurring.
The result is minimization by scale-continuation

The edge-energy functional is $$E_{line} =-(G_\sigma * \nabla^2I)^2$$
where \\(G_\sigma\\) is a Gaussian of 標準差
Minima of this functional lie on zero-crossings of $$G_\sigma * \nabla^2I$$ which define edges in the MarrHildreth theory
在snake上加上energy term 代表 snake is attracted to zero-crossings,但還是被自己的smoothness限制

## Termination Functional 
In order to find terminations of line segments and corners,we use the curvature of level lines in a slightly smoothed image.
Let $$C(x,y)=G_\sigma(x,y)*I(x,y)$$ be a slightly smoothed version of  the image
Let $$\theta=tan^{-1}(\frac{C_y}{C_x})$$ be the gradient angle
Let $$n=(cos \theta,sin \theta)$$ and $$n_\perp=(-sin \theta,cos \theta)$$ be unit vectors along and perpendicular to the gradient direction.
Then the curvature of the level contours in $$C(x,y)$$ can be written
\begin{split}
E_{term} & = \frac{\partial \theta}{\partial n_{\perp}} \\
& = \frac{\partial^2 C/\partial n^2_{\perp}}{\partial C/\partial n_{\perp}} \\
&=\frac{C_{yy}C^2_x-2C_{xy}C_xC_y+C_{xx}C^2_y}{(C^2_x+C^2_y)^{3/2}}
\end{split}

合併$$E_{edge}$$和$$E_{term}$$ snakes is attracted to edges and terminations


# Optimization through gradient descent
 Gradient descent minimization is one of the simplest optimizations which can be used to minimize snake energy
$${\bar  v}_{i}\leftarrow {\bar  v}_{i}+F_{snake}({\bar  v}_{i})$$
\\(F_{snake}({\bar  v}_{i})\\) is the force on the snake

$$F_{snake}({\bar  v}_{i})=-\nabla E_{snake}({\bar  v}_{i})=-{\Bigg (}w_{internal}\nabla E_{internal}({\bar  v}_{i})+w_{external}\nabla E_{external}({\bar  v}_{i}){\Bigg )}$$

# Stereo and Motion
## Stereo
Snakes can also be applied to the problem of stereo matching
In stereo,if two contours correspond ,then the disparity should vary slowly along the contour unless the contour rapidly recedes in depth

Psychophysical evidence of a disparity gradient limit in human stereopsis indicates that the human visual system at least to some degree assumes that disparities do not change too rapidly with space.
This constraint can be expressed in an additional energy functional for a stereo snake:
$$E_{stereo}=(v_s^L(s)-v_s^R(s))^2$$
\\(v^L(s)\\),\\(v^R(s)\\)分別是left contour 和 right contour


Since the disparity smoothness constraint is applied along contours, it shares a strong similarity with Hildreth’s smoothness constraint for computing optic flow.
This constraint means that during the process of localizing a contour in one eye, information about the corresponding contour in the other eye is used.
In stereo snakes, the stereo match actually affects the detection and localization of the features on which the match is based.
This differs importantly, for example, from the Marr-Poggio stereo theory in which the basic stereo matching primitive zero-crossings always remain unchanged by the matching process.

## Motion
If the feature then begins to move slowly, the snake will simply track the same local minimum.
Edge-attracted snakes were initialized by hand on the speaker’s lips in the first frame.
After that, the snakes tracked the lip movements automatically.
Give the snake mass ,then it will predict next position based on its previous velocity

# References and Links