---
layout: project
title: Live Video Masking with Gesture Input
date: March 16, 2016
image: nate.png
---

# Demo
<iframe width="560" height="315" src="https://www.youtube.com/embed/Z36AU7aYf0A" frameborder="0" allowfullscreen></iframe>

# Partner
This project was completed jointly with [Matt Cruz](http://matthewcruz.github.io/portfolio/projects/20160315ImageProject1/). THis writeup will focus primarly on my personal contribution to this project. 

# Overview
The goal for this project was to design a system that could detect a moving finger and use that as the cursor for a virtual drawing interface. Idealy the system would work in almost any lighting conditions and with almost any background environment. The interface would let a person draw on the live video feed and would use different finger positions.

# Process
Once the foreground is extracted and thresholded, what is left should mostly be the hand. However there still may be some noise from the background. In order to detect the hand, some kind of shape detection is needed. We used an OpenCV function, cv2.findContours() to detect contour lines in the image. By assuming both that the hand is is larger than any background noise, and is is a fairly constant size in the image, The function can be used to find the shape of the hand while ignoring background noise. The cv2.findContours() was set to find the contour in the image with the largest area using the argument cv2.RETR_EXTERNAL, and only displayed said contour if the area was greater than a threshold size that was experimentally determined. This allows the cv2.findContours() to reliably find the shape of the outside of the hand, but reject background noise leftover from foreground extraction. 

After detecting the hand contours, the location of the fingertips is still needed. To this end, another OpenCV function was utilized. The function cv2.convexHull() uses an algorithm to detect convexity defects in a given curve. If the curve is supposed to be purely convex, these defects are points where the curve temporarily bulges inward. The function returns a set of points on the curve that, if connected, would make the curve perfectly convex. These points are used to detect fingertips because on a curve fitting the outside of a hand, fingers create convexity defects. It was found that if fingers were outstretched at an angle the camera could see (e.g. visibly outstretched away from the hand), the function would always return points near the fingertips. As long as the finger position remained unchanged, these points were fairly consistent frame to frame, and thus were used to track the tips of the fingers. 

To detect different gestures, more analysis of these points was necessary. Different finger positions were analyzed with respect to the location and density of these convexity defect points, and it was found that a clear difference existed between an open palm gesture and a pointing gesture. Specifically, with an open palm, a few convexity defect points were spaced out along each fingertip, as well as along irregular curves in the arm. With a pointing gesture though, a large number of points was found concentrated at the tip of the pointing finger. 

This fact was used to differentiate between the two gestures. A simple algorithm measured the cartesian distance between each convexity defect point and its four neighbors. If the distance to each of the neighbors is under an experimentally determined threshold, the point was the tip of a finger making a pointing gesture. In this case, neighbors refers to the points immediately in front of and behind the tested point in the returned list of points. Because the list follow the contour of the hand in the clockwise direction, edge cases are handled by checking points at the opposite ends of the list as these would be neighbors if the contour formed a closed loop. 

By using this detection functionality, the system could reliably identify a pointing gesture and reject an open hand gesture. This distinction was then used for the drawing interface. If a pointing gesture was detected, the point at the fingertip was tracked. This point was also entered into a custom function which drew a small colored circle centered on the returned coordinates. This circle was specifically drawn on a blank mask the same dimension as the video feed. This mask was later overlaid with the raw video feed and displayed as the final system output. This drawing function was specifically disabled if for the open palm gesture so the user could use that gesture as a “stop drawing” command, allowing for the effective functionality of analogously picking up the virtual pen. 

# Outputs
Video feed after Color thresholding and background extraction

<img src="/public/images/bg.png" alt="After Color thresholding and background extraction" style="width: 400px;"/>

This is what is returned after the initial processing (my partner’s work primarily). The hand is mostly intact and isolated from the background. However, there is some leftover background noise. How much noise remains is largely a function of lighting conditions. 

Raw video with detected Contour (blue) and convexity defect points (red) displayed:

<img src="/public/images/bg_contour.png" alt="Detected Contour and Convexity Points" style="width: 400px;"/>

After the cv2.findContours() returns a suitably large contour and cv2.convexHull() finds the convexity defect points, an intermediary visualization step returns the raw video feed with the colour displayed in blue and the convexity defect points displayed in red. If no hand was in the frame, no red points or blue lines would show up on the screen because of the area threshold in place. 

Demonstration of high density of red points at fingertip during pointing gesture:

<img src="/public/images/contour_finger.png" alt="Fingertip Characterization" style="width: 400px;"/>

This demonstrates what we found by analyzing the convexity defect points of different gestures. For this pointing gesture, a large number of red points were displayed in a highly concentrated spot at the tip of the finger. This fact is used to detect this gesture specifically and track the fingertip. 

Black Circle follows tracked fingertip: 

<img src="/public/images/finger_detect.png" alt="Fingertip Detected" style="width: 400px;"/>

Another intermediary step displayed a black circle at the detected fingertip. It only displayed if the pointing gesture was used, and it followed the fingertip accurately if the hand was moved in the frame. 

Demonstrating Open Palm gesture not triggering tracked fingertip (black circle):

<img src="/public/images/no_finger_detect.png" alt="Fingertip Not Detected" style="width: 400px;"/>

This is the same step as the previous picture. The lack of black circle specifically demonstrated that the system can differentiate between this open palm gesture and the previous pointing gesture. This functionality is how the drawing is turned “on” and “off”. 

Writing Example: 

<img src="/public/images/hi.png" alt="Writing with the system" style="width: 400px;"/>

This is a demonstration that a user can write successfully on the screen with this system. If the open hand gesture had been falsely identified and tracked, there would be colored spots on the hand. 

# Files

## Github

[Here is the Github repo for the project](https://github.com/ncorwin/image_analisys_final.git)
