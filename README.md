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

[image1]: ./undistorted_images/undistorted_with_corners.png "Undistorted with Corners"
[image2]: ./output_images/undistorted.png "distortion corrected"
[image3]: ./output_images/combined_threshold.png "Binary Example"
[image4]: ./output_images/lane_lines_top_down.png "Warp Example"
[image5]: ./output_images/sliding_windows.png "Sliding Windows"
[image6]: ./output_images/past_lane_lines.png "If Previous"
[image7]: ./output_images/filled_lane.png "Filled Lane"
[video1]: ./project_video_output.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the in the IPython notebook under the headers "Creating objpoints and imgpoints" and "Calibrating and Distortion Correction"

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![undistorted with corners][image1]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

Here is an example of an image which had distortion correction applied to it:
![distortion corrected][image2]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used a combination of different color  thresholds to generate a binary image. In the iPython notebook, these are under a heading titled "Color Thresholding". Here's an example of my output for this step.

![color threshold][image3]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes a function called `perspective_transform()`, under the corresponding header of the IPython notebook.  The `perspective_transform()` function takes as inputs an image (`img`), as well as source (`src`) and destination (`dst`) points.  I chose the hardcode the source and destination points in the following manner:

```python
w,h = 1280,720
x,y = 0.5*w, 0.8*h
src = np.float32([[200./1280*w,720./720*h],
              [453./1280*w,547./720*h],
              [835./1280*w,547./720*h],
              [1100./1280*w,720./720*h]])
dst = np.float32([[(w-x)/2.,h],
              [(w-x)/2.,0.82*h],
              [(w+x)/2.,0.82*h],
              [(w+x)/2.,h]])
```

This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 200, 720      | 320, 720      | 
| 453, 547      | 320, 590      |
| 835, 547      | 960, 590      |
| 1100, 720     | 960, 720      |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![perspective transform][image4]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

In order to identify the lane-line pixels, I first made a histogram identifying the pixel counts in the warped image. This allowed me to identify the two peaks which corresponded to the two lane lines. Given these peaks, I used a rectangle centered around each of the peaks, and moved the window up along the images with the lane lines in the center. I was then able to apply a polynomial fit to this, by keeping track of the lane indices as I moved up the image. Here is an example:

![sliding window][image5]

If this method has already been applied to the previous image frame, we can use a slightly altered method given the previous frame's image to identify the lane-lines. Here is the corresponding image:

![if previous][image6]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

I calculated the radius of curvatire and the vehicle position in the function `fill_lane()`.

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in in conjunction with the function I wrote to calculate the radius of curvature. This is under the "Fill Lane" header. Here is an example of my result on a test image:

![fill lane][image7]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./project_video.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

In my implementation, I first spent a significant amount of time trying different color and gradient thresholds. I ended up not actually using gradient thresholding, but used color thresholding with two different color spaces. There was one test image which was particularly difficult to identify the lane lines with, because that image had a bright patch followed by a large shadow cast by a tree. It is situations like these where my pipeline is most likely to fail, namely situations in which there is a quick change between the surroundings. The best way to improve on this would be to experiment in greater depth with either other color thresholds, or by adding gradient thresholding in as well.
