---
title: Detecting Faces in Images A Survey
tags:
  - Face Detection
  - Face Recognition
  - Object Recognition
categories:
  - Computer Vision
date: 2018-08-12 21:57:13
---
論文筆記(待續)

# Introduction
A first step of any face processing system is detecting the locations in images where faces are present.
However, face detection from a single image is a challenging task because of variability in scale, location, orientation (up-right, rotated), and pose (frontal, profile).
Facial expression, occlusion, and lighting conditions also change the overall appearance of faces

We now give a definition of face detection: Given an arbitrary image, the goal of face detection is to determine whether or not there are any faces in the image and, if present, return the image location and extent of each face.
The challenges associated with face detection can be attributed to the following factors:
* **Pose** The images of a face vary due to the relative camera-face pose (frontal, 45 degree, profile, upsidedown), and some facial features such as an eye or the	nose may become partially or wholly occluded.
* **Presence or absence of structural components** Facial features such as beards, mustaches, and glasses may or may not be present and there is a great deal of variability among these components including shape, color, and size.
* **Facial expression** The appearance of faces are directly affected by a person’s facial expression
* **Occlusion** Faces may be partially occluded by other objects. In an image with a group of people, some faces may partially occlude other faces
* **Image orientation** Face images directly vary for different rotations about the camera’s optical axis.
* **Imaging conditions** When the image is formed, factors such as lighting (spectra, source distribution and intensity) and camera characteristics (sensor response, lenses) affect the appearance of a face.

*Face localization* aims to determine the image position of a single face; this is a simplified detection problem with the assumption that an input image contains only one face

The goal of *facial feature detection* is
to detect the presence and location of features, such as eyes, nose, nostrils, eyebrow, mouth, lips, ears, etc., with the assumption that there is only one face in an image

*Face recognition *or *face identification *compares an input image(probe) against a database (gallery) and reports a match, if any
The purpose of *face authentication* is to verify the claim of the identity of an individual in an input image , while face tracking methods continuously estimate the location and possibly the orientation of a face in an image sequence in real time
*Facial expression recognition* concerns identifying the affective states (happy, sad, disgusted, etc.) of humans
Evidently, face detection is the first step in any automated system which solves the above problems.
It is worth mentioning that many papers use the term “face detection,” but the methods and the experimental results only show that a single face is localized in an input image.
In this paper, we differentiate face detection from face localization since the latter is a simplified problem of the former.
Meanwhile, we focus on face detection methods rather than tracking methods

# DETECTING FACES IN A SINGLE IMAGE
1. **Knowledge-based methods** These rule-based methods encode human
knowledge of what constitutes a typical face.
Usually, the rules capture the relationships between facial features.
These methods are designed mainly for face localization.
2. **Feature invariant approaches** These algorithms aim
to find structural features that exist even when the
pose, viewpoint, or lighting conditions vary, and then use the these to locate faces.
These methods are designed mainly for face localization.
3. **Template matching methods** Several standard patterns
of a face are stored to describe the face as a whole or the facial features separately.
The correlations between an input image and the stored patterns are
4. **Appearance-based methods** In contrast to template
matching, the models (or templates) are learned from
a set of training images which should capture the
representative variability of facial appearance.
These learned models are then used for detection.
These methods are designed mainly for face detection.