#**Advanced Lane Finding**

## Kerem Par

### kerempar@gmail.com

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

[image1]: ./output_images/undistort_output.png =700x200 "Undistorted"
[image2]: ./output_images/road_transformed.png =700x200 "Road Transformed"
[image3]: ./output_images/binary_warped_images.png =700x400 "Binary Example"
[image4]: ./output_images/warped_straight_lines.png =700x200 "Warp Example"
[image6]: ./output_images/example_output.png =500x250 "Output"
[image7]: ./output_images/sliding_window_polyfit.png =400x200 "Sliding Window"
[image8]: ./output_images/polyfit_from_previous_frame.png =400x200 "Polyfit from Previous Frame"
[image9]: ./output_images/histogram.png =400x200 "Histogram"
[image10]: ./output_images/binary_warped_image.png =700x200 "Warp Example2"
[video1]: ./project_video_output.mp4 "Video"


## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the first code cell (under `Camera Calibration` heading) of the IPython notebook located in [Advanced-Lane-Lines.ipynb](./Advanced-Lane-Lines.ipynb).  

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection (detection for some of these images were not successful because the specified number of chessboard corners were not found).  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to one of the test images using the `cv2.undistort()` function and obtained this result: 

![alt text][image1]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

To demonstrate this step, I will describe how I apply the distortion correction to one of the test images like this one :

![alt text][image2]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I tried various Sobel gradient and color space thresholds (RGB, HLS, HSV, LAB) with test images in a serious of code cells in the notebook. I finally used a combination of color  thresholds (HLS L-channel and LAB B-channel thresholds) to generate a binary image (thresholding steps at `Pipeline (Test Images)` code cell of the notebook). Here's an example of my output for this step for two of the test images.

![alt text][image3]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes a function called `warper()`, which appears in `Perspective Transform` code cell of the IPython notebook). The `warper()` function takes as inputs an image (`img`), as well as source (`src`) and destination (`dst`) points.  I chose to hardcode the source and destination points by picking four points in a trapezoidal shape that would represent a rectangle when looking down on the road from above in the following manner (taken from the code used during the lesson):

```python
src = np.float32([(575,464),
                  (707,464), 
                  (258,682), 
                  (1049,682)])
dst = np.float32([(450,0),
                  (w-450,0),
                  (450,h),
                  (w-450,h)])
```

This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 575, 464      | 450, 0        | 
| 707, 464      | 830, 0      |
| 258, 682     | 450, 720      |
| 1049, 682      | 830, 720        |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.


![alt text][image4]

![alt text][image10]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

Then I used sliding window technique to detect left and right lane-line pixels. I used histogramming for the lower half of binary warped image to detect peaks as starting points. Then used sliding window search to find lane pixels located around window center points. And then fit my lane lines with a 2nd order polynomial
in the `polyfit_using_sliding_window_search()` function in the `Locate Lane Lines Using Sliding Window Search` code cell of the notebook:

![alt text][image9]

![alt text][image7]

I also implemented `polyfit_using_prev_fit()` function to reuse the fit from the previous frame for easily detect lanes in the next frame in the `Locate Lane Lines Using Fit from Previous Frame` code cell of the notebook:

![alt text][image8]

The idea here is that once we have a high-confidence detection, using that to inform the search for the position of the lines in subsequent frames of video.

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

I calculated curvature radius and the distance of the vehicle from the lane center in `measure_curv_and_center_dist()` function in the `Measuring Curvature` code cell in the notebook. I used the piece of code given in the lesson. I assumed that the lane projected in the images is about 30 meters long and 3.7 meters wide. I calculated the distance of the vehicle with respect to center as the image x midpoint - mean of l_fit and r_fit intercepts.

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in the `draw_data()` function in the `Draw Curvature Radius and Distance` code cell in the notebook.  Here is an example of my result on a test image:

![alt text][image6]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./project_video_output.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

Here I'll talk about the approach I took, what techniques I used, what worked and why, where the pipeline might fail and how I might improve it if I were going to pursue this project further.  

The project video includes both white and yellow lane lines and different lighting conditions like shadows, both dark and bright bakground colors. For thresholding I have tested various gradient and color space thresholds (RGB, HLS, HSV, LAB) with test images. I have clearly observed that RGB thresholding works best on white lane pixels, but does not work that well for images that include varying lighting conditions and lane colors like yellow. I could not find Sobel thresholding methods were not successful either when tried with project video. As already emphasized during the lessons, the S channel of HLS color scheme did a fairly robust job of picking up the lines under different color and contrast conditions, however it still caused issues in some places throughout the video. On the other hand, I noticed that the B channel of LAB color scheme is very suitable for detecting yellow lines (the yellow/blue opponent colors are represented along the B axis, with blue at negative b values and yellow at positive b values). So, I finally tried a combination of HLS L channel and LAB B channel thresholding to handle all the cases of the video successfully.  

The pipeline will probably fail in the challenge videos where lighting and contrast conditions are different. Probably they will require different thresholding combinations or different threshold values for thresholding steps.

