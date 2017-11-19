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

#### The code and implementation of the solution step by step can be found in the IPython notebook [P4.ipynb](./P4.ipynb).

---

### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!


### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the cells 2 to 6 (including tests) of the IPython notebook located in [P4.ipynb](./P4.ipynb).  

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  The variable `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image. `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection, this mechanism is implemented in the method `getObjImgPointsAndShape()` and was applied to all the images in the directory `camera_cal`.

Here is an example of the detection of the chess board corners:

![alt text][image1]

In the method `calibrateCamera()`, I use `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function, in order to use these values in the further method I created the class `Calibration`.

Here are is an example of the application of the distortion correction applied to a chessboard image:

![alt text][image2]


### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

The following is an example of the application of distortion correction to one of the test chess board images, the difference should be evident noticing how the curve lines are transformed to straight ones:

![alt text][image2]

And this is an example applied to a road image, a little less evident, but looking at the reflection in windshield it can be seen how a curved line is transformed straightened.

![alt text][image3]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used a combination of color and gradient thresholds to generate a binary image, first images are converted to HLS color space, the channel S is separated to do a thresholding by color magnitude, to do the gradient thresholding the channel L used and the derivarive applied to it. The code can be found in cells 7 and 8 of [P4.ipynb](./P4.ipynb), specifically the implementation is in the method `getThresholdedBinaryImage`.

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

This points are plotted in the following image:

![alt text][image5]

The code for my perspective transform includes a function called `warper()`, which appears in lines 1 through 8 in the file `example.py` (output_images/examples/example.py) (or, for example, in the 3rd code cell of the IPython notebook).  The `warper()` function takes as inputs an image (`img`), as well as source (`src`) and destination (`dst`) points.  I chose the hardcode the source and destination points in the following manner:

```python
src = np.float32(
    [[(img_size[0] / 2) - 55, img_size[1] / 2 + 100],
    [((img_size[0] / 6) - 10), img_size[1]],
    [(img_size[0] * 5 / 6) + 60, img_size[1]],
    [(img_size[0] / 2 + 55), img_size[1] / 2 + 100]])
dst = np.float32(
    [[(img_size[0] / 4), 0],
    [(img_size[0] / 4), img_size[1]],
    [(img_size[0] * 3 / 4), img_size[1]],
    [(img_size[0] * 3 / 4), 0]])
```


This resulted in the following source and destination points:

| Source        | Destination   |
|:-------------:|:-------------:|
| 585, 460      | 320, 0        |
| 203, 720      | 320, 720      |
| 1127, 720     | 960, 720      |
| 695, 460      | 960, 0        |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![alt text][image6]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

Then I did some other stuff and fit my lane lines with a 2nd order polynomial kinda like this:

![alt text][image5]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

I did this in lines # through # in my code in `my_other_file.py`

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in lines # through # in my code in `yet_another_file.py` in the function `map_lane()`.  Here is an example of my result on a test image:

![alt text][image6]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./project_video.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

Here I'll talk about the approach I took, what techniques I used, what worked and why, where the pipeline might fail and how I might improve it if I were going to pursue this project further.  
