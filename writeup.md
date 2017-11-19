## Advanced Lane Finding

---

**Advanced Lane Finding Project**

The goals / steps of this project are the following:

* Compute the camera calibration matrix and distortion coefficients given a set of chessboard images.
* Apply a distortion correction to raw images.
* Use color transforms, gradients, etc., to create a thresholded binary image.
* Apply a perspective transform to rectify binary image ("birds-eye view").
* Detect lane pixels and fit to find the lane boundary.
* Determine the curvature of the lane and vehicle position with respect to center.
* Warp the detected lane boundaries back onto the original image.
* Output visual display of the lane boundaries and numerical estimation of lane curvature and vehicle position.

[//]: # (Image References)

[image1]: ./output_images/IdentifiedChessCorners.png "Identification of chess corners"
[image2]: ./output_images/UndistortedChessBoard.png "Undistortion of chess board"
[image3]: ./output_images/UndistortedRoadImage.png "Undistortion of road image"
[image4]: ./output_images/ThresholdedBinaryImage.png "Thresholding of binary image"
[image5]: ./output_images/PerspectivePoints.png "Perspective points for reference"
[image6]: ./output_images/WarpedImage.png "Rectification (warping) of road image"
[image7]: ./output_images/UnwarpedImage.png "Unwarping of road image"
[image8]: ./output_images/BirdsEyeProcess.png "Process to get a binary birds-eye image"
[image9]: ./output_images/WarpedBinaryImage.png "Getting birds-eye view from road image"
[image10]: ./output_images/LaneDetectionProcess.png "Lane detection process"
[image11]: ./output_images/LaneCurvatureDetermination.png "Lane curvature determination"
[image12]: ./output_images/LaneDetectionSample.png "Example of lane detection process in road image"
[video]: ./result.mp4 "Video"


## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

#### The code and implementation of the solution step by step can be found in the IPython notebook [P4.ipynb](./P4.ipynb) or the HTML file [P4.html](./P4.html).

---

### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!


### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the cells 2 to 6 (including tests) of the IPython notebook located in [P4.ipynb](./P4.ipynb).  

I start by preparing "object points", which will be the (x, y, z) coordinates of the chess board corners in the world. Here I am assuming the chess board is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  The variable `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chess board corners in a test image. `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection, this mechanism is implemented in the method `getObjImgPointsAndShape()` and was applied to all the images in the directory `camera_cal`.

Here is an example of the detection of the chess board corners:

![alt text][image1]

In the method `calibrateCamera()`, I use `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function, in order to use these values in the further methods I created the class `Calibration`.

Here are is an example of the application of the distortion correction applied to a chessboard image:

![alt text][image2]


### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

The following is an example of the application of distortion correction to one of the test chess board images, the difference should be evident noticing how the curve lines are transformed to straight ones:

![alt text][image2]

And this is an example applied to a road image, a little less evident, but looking at the reflection in windshield it can be seen how a curved line is straightened.

![alt text][image3]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used a combination of color and gradient thresholds to generate a binary image, first images are converted to HLS color space, the channel S is separated to do a thresholding by color magnitude, to do the gradient thresholding the channel L used and the derivative applied to it. The code can be found in cells 7 and 8 of [P4.ipynb](./P4.ipynb), specifically the implementation is in the method `getThresholdedBinaryImage()`.

Here is an example of the output for this step:

![alt text][image4]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code of the perspective transform's implementation can be found in cells 9 to 14 of [P4.ipynb](./P4.ipynb). In order to establish the base for doing the perspective transformation, I chose to hardcode the source and destination points as follows:

| Source        | Destination   |
|:-------------:|:-------------:|
| 520, 500      | 320, 500      |
| 765, 500      | 980, 500      |
| 200, 720      | 320, 720      |
| 1100, 720     | 980, 720      |

These points are plotted in the following image:

![alt text][image5]

I created a class called `PerspectiveTransform` in order to store the perspective matrix and the inverse perspective matrix, the last one to be used later in the step to draw the identified lane in the image/video. The code for my perspective transform includes a function called `getPerspectiveTransform()` which takes as inputs the source (`src`) and destination (`dst`) points to calculate the perspective matrices and store them in an object of the mentioned class. Additionally they are two methods for warping (rectifying) or unwarping and image using a `PerspectiveTransform` object.

I verified that my perspective transformation was working as expected by warping an image of a straight road section and verifying that the lines appear parallel in the warped image:

![alt text][image6]

Also, I verified that the inverse perspective transformation is returning the image to the original state, well, actually just the section surrounding the lane:

![alt text][image7]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

First, I prepare the image by undistorting, applying color/gradient thresholds and warping, the result is a bird-eye like image that is going to be used to detect the lane lines:

![alt text][image8]

To identify the pixels belonging to the lane lines, I used the method of window fitting given a defined window size of 80x80, and applying convolution to look for the pixels by vertical layers. I also incorporated a look ahead mechanism, if a previously fitted polynomial for a lane line is given, this is used to calculate the initial position on the bottom of the image to start looking for the line pixels.  The code can be found in cells 18 to 20 of [P4.ipynb](./P4.ipynb), the base method is `findWindowCentroids()` which returns the collection of centroids that represent the section corresponding to the line, the following is an example of the lane lines detection process:

![alt text][image10]

The detection mechanism is grouped together with the curvature calculation, and some mechanisms for fault tolerance (like sanity check and buffering previously found lines) further in the code, cells 23 to 26 of [P4.ipynb](./P4.ipynb).


#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

Lane curvature determination is implemented in cells 21 and 22 of [P4.ipynb](./P4.ipynb), for calculating the radius of curvature given a second degree polynomial (i.e. the ones used for the lane lines), I applied the following formula in the method `calculateCurvatureOfLaneLine()`, where the polynomial is of the form `x = A * y**2 + B * y + C`:
```
c = ((1 + (2 * A * y + B)**2)**1.5) / np.absolute(2 * A)
```
The value of `y` was the high of the image multiplied by the ratio of meters per pixel, in order to get the radius of the curvature in meters.

Here is an example for the curvature detection:

![alt text][image11]

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

This is an example plotting back the detected lane into the image (inverse perspective transformation was used for this), the method in charge to do it is `drawFoundLaneOverImage()` which is found in the cell 24 of [P4.ipynb](./P4.ipynb).

![alt text][image12]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here is the [link to my video result](./result.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

As suggested during the lessons, I implemented some fault tolerance mechanisms in case that the lane line can not be detected with confidence, these include:
* Buffering: The last N previous lines are considered and the ones reliably detected are stored in the buffer. This has the objective of providing an approximation in case that the lane in an image can not be detected.
* Sanity Checking: Used to determine if a lane is "reliably" detected, basically it checks that the retrieved lines are roughly parallel, their separation is around (3.7m the standard for the US highways) and they are not to different to the lines detected before.

There is a couple of sections where the change of color on the pavement and the presence of shadows make it difficult to detect the lines (even for the human eye), in this case the buffer offers a good mechanism to infer the line. I noticed that large buffers compensate the lack of identification better, however that makes less flexible to adapt to changing circumstances like a narrower curve ahead or a bumpy section. So part of the challenge was to find the balance between flexibility and recovering for lack of detection, the amount that seemed to work better was a buffer size of 50.
