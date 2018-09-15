---
title: SSD Single Shot MultiBox Detector
tags:
  - Real-time Object Detection
  - Convolutional Neural Network
  - Single Shot Detector
categories:
  - Computer Vision
date: 2018-09-01 14:03:17
---

論文筆記(待續) 

# Abstract
Our approach, named SSD, discretizes the output space of
bounding boxes into a set of default boxes over different aspect ratios and scales per feature map location
At prediction time, the network generates scores for the
presence of each object category in each default box and produces adjustments to
the box to better match the object shape
Additionally, the network combines predictions
from multiple feature maps with different resolutions to naturally handle
objects of various sizes. 
SSD is simple relative to methods that require object
proposals because it completely eliminates proposal generation and subsequent
pixel or feature resampling stages and encapsulates all computation in a single
network


# Introduction



Current state-of-the-art object detection systems are variants of the following approach:
hypothesize bounding boxes, resample pixels or features for each box, and apply a highquality classifier

While accurate, these approaches have been too computationally intensive for embedded
systems and, even with high-end hardware, too slow for real-time applications.

Often detection speed for these approaches is measured in seconds per frame (SPF),
and even the fastest high-accuracy detector, Faster R-CNN, operates at only 7 frames
per second (FPS)

This paper presents the first deep network based object detector that does not resample
pixels or features for bounding box hypotheses and and is as accurate as approaches
that do

The fundamental improvement in
speed comes from **eliminating bounding box proposals and the subsequent pixel or feature
resampling stage**

Our improvements include using a small convolutional filter to predict object
categories and offsets in bounding box locations, using separate predictors (filters) for
different aspect ratio detections, and applying these filters to multiple feature maps from
the later stages of a network in order to perform detection at multiple scales

With these
modifications—especially using multiple layers for prediction at different scales—we
can achieve high-accuracy using relatively low resolution input, further increasing detection
speed

## Summarize

* We introduce SSD, a single-shot detector for multiple categories that is faster than
the previous state-of-the-art for single shot detectors (YOLO), and significantly
more accurate, in fact as accurate as slower techniques that perform explicit region
proposals and pooling (including Faster R-CNN).


* The core of SSD is predicting category scores and box offsets for a fixed set of
default bounding boxes using small convolutional filters applied to feature maps.

* To achieve high detection accuracy we produce predictions of different scales from
feature maps of different scales, and explicitly separate predictions by aspect ratio.

* These design features lead to simple end-to-end training and high accuracy, even
on low resolution input images, further improving the speed vs accuracy trade-off.

* Experiments include timing and accuracy analysis on models with varying input
size evaluated on PASCAL VOC, COCO, and ILSVRC and are compared to a
range of recent state-of-the-art approaches.

# SSD
## Model
The SSD approach is based on a feed-forward convolutional network that produces
a fixed-size collection of bounding boxes and scores for the presence of object class
instances in those boxes, followed by a non-maximum suppression step to produce the
final detections

**Multi-scale feature maps for detection**
We add convolutional feature layers to the end
of the truncated base network. These layers decrease in size progressively and allow
predictions of detections at multiple scales. The convolutional model for predicting
detections is different for each feature layer

**Convolutional predictors for detection**
Each added feature layer (or optionally an existing
feature layer from the base network) can produce a fixed set of detection predictions
using a set of convolutional filters. These are indicated on top of the SSD network
architecture in Fig. 2. For a feature layer of size m × n with p channels, the basic element
for predicting parameters of a potential detection is a 3 × 3 × p small kernel
that produces either a score for a category, or a shape offset relative to the default box
coordinates. At each of the m × n locations where the kernel is applied, it produces an
output value. The bounding box offset output values are measured relative to a default
box position relative to each feature map location

**Default boxes and aspect ratios**
We associate a set of default bounding boxes with
each feature map cell, for multiple feature maps at the top of the network. The default
boxes tile the feature map in a convolutional manner, so that the position of each box
relative to its corresponding cell is fixed. At each feature map cell, we predict the offsets
relative to the default box shapes in the cell, as well as the per-class scores that indicate
the presence of a class instance in each of those boxes. Specifically, for each box out of
k at a given location, we compute c class scores and the 4 offsets relative to the original
default box shape. This results in a total of (c + 4)k filters that are applied around each
location in the feature map, yielding (c + 4)kmn outputs for a m × n feature map. For
an illustration of default boxes, please refer to Fig. 1. Our default boxes are similar to
the anchor boxes used in Faster R-CNN [2], however we apply them to several feature
maps of different resolutions. Allowing different default box shapes in several feature
maps let us efficiently discretize the space of possible output box shapes.

## Training
The key difference between training SSD and training a typical detector that uses region
proposals, is that ground truth information needs to be assigned to specific outputs in
the fixed set of detector outputs
Once this assignment is determined, the loss function and back propagation are applied endto-end.
Training also involves choosing the set of default boxes and scales for detection
as well as the hard negative mining and data augmentation strategies

** Matching strategy **
During training we need to determine which default boxes correspond
to a ground truth detection and train the network accordingly. For each ground
truth box we are selecting from default boxes that vary over location, aspect ratio, and
scale. We begin by matching each ground truth box to the default box with the best
jaccard overlap (as in MultiBox [7]). Unlike MultiBox, we then match default boxes to
any ground truth with jaccard overlap higher than a threshold (0.5). This simplifies the
learning problem, allowing the network to predict high scores for multiple overlapping
default boxes rather than requiring it to pick only the one with maximum overlap

**Training objective**
The overall objective loss
function is a weighted sum of the localization loss (loc) and the confidence loss (conf):
$$
L(x, c, l, g) = \frac{1}{N}(L_{conf}(x,c)+\alpha L_{loc(x,l,g)})
$$
where N is the number of matched default boxes
If N = 0, wet set the loss to 0
The localization loss is a Smooth L1 loss [6] between the predicted box (l) and the ground
truth box (g) parameters. Similar to Faster R-CNN [2], we regress to offsets for the
center (cx, cy) of the default bounding box (d) and for its width (w) and height (h).
$$ L_{loc}(x,l,g)=\sum^N_{i\in Pos}\sum_{m\in {cx,cy,w,h}}x^k_{ij}smooth_{L1}(l^m_{i}-\hat g^m_{j}) $$

$$ \hat g^{cx}_j=(g^{cx}_j-d^{cx}_i)/d^{w}_i \\  \hat g^{cy}_j=(g^{cy}_j-d^{cy}_i)/d^{h}_i $$
$$ \hat g^{w}_j=log(\frac{g^w_j}{d^w_i}) \\  \hat g^{h}_j=log(\frac{g^h_j}{d^h_i}) $$


The confidence loss is the softmax loss over multiple classes confidences (c).
$$ L_{conf}(x, c) =-\sum ^N_{i \in Pos}x^p_{ij}log(\hat c^p_i)-\sum_{i \in Neg} log(\hat c^0_i)$$
where, $$ \hat c^p_i=\frac{exp(c^p_i)}{\sum_p exp(c^p_i)} $$
and the weight term \\(\alpha\\) is set to 1 by cross validation

** Choosing scales and aspect ratios for default boxes**
To handle different object scales,
some methods [4,9] suggest processing the image at different sizes and combining the
results afterwards
However, by utilizing feature maps from several different layers in a
single network for prediction we can mimic the same effect, while also sharing parameters
across all object scales
Previous works [10,11] have shown that using feature maps
from the lower layers can improve semantic segmentation quality because the lower
layers capture more fine details of the input objects
Similarly, [12] showed that adding global context pooled from a feature map can help smooth the segmentation results

Motivated by these methods, we use both the lower and upper feature maps for detection

Feature maps from different levels within a network are known to have different
(empirical) receptive field sizes
Fortunately, within the SSD framework, the default boxes do not necessary need to correspond to the actual receptive fields of each
layer. We design the tiling of default boxes so that specific feature maps learn to be
responsive to particular scales of the objects
Suppose we want to use m feature maps
for prediction. The scale of the default boxes for each feature map is computed as:
$$ S_k=S_{min}+\frac{S_{max}-S_{min}}{m-1}(k-1),k\in [1,m] $$

** Hard negative mining **
After the matching step, most of the default boxes are negatives,
especially when the number of possible default boxes is large
Instead of
using all the negative examples, we sort them using the highest confidence loss for each
default box and pick the top ones so that the ratio between the negatives and positives is
at most 3:1. We found that this leads to faster optimization and a more stable training

