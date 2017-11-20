# **Advanced Lane Finding Project**

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

[image1]: output_images/undistorted_calibration1.jpg "Undistorted"
[image7]: output_images/undistorted_straight_lines1.jpg "Undistorted Test Frame"
[image2]: output_images/binary_straight_lines1.jpg "Binary Example"
[image3]: output_images/binary_warped_straight_lines1.jpg "Warp Example"
[image4]: output_images/line_fits_straight_lines1.jpg "Sliding Windows Visualization"
[image5]: output_images/warped_line_fits_straight_lines1.jpg "Fit Visualization"
[image6]: output_images/final_straight_lines1.jpg "Output Visualization"
[video1]: project_video_output.mp4 "Video"

## Rubric Points

#### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  A large part of the code has been copied from various parts of the lesson, with some variation and improvement (hopefully) on my part.  I've tried to indicate where code from the lesson has been pasted where applicable.

---


### Camera Calibration

(Code for this step is in the camera_calibration cell of advanced-lane-lines.ipynb)

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result:

![alt text][image1]
![alt text][image7]

Per the first submission, I did leave out the undistortion step in the pipeline, but I have corrected it in this submission.

### Pipeline (single images)

(Code for this step is in the Color and gradient cell of the notebook)

In this step I executed the main pipeline for processing of the initial images.  I first read in the image and convert it to the HLS color space.  The l and s channel seem to be the most useful for this application, so I continue with those.  I take a horizontal Sobel transform of the l_channel in order to accentuate vertical lines.  Then I apply a threshold in order to eliminate noise.  I also applied a threshold to the s_channel, and then combined these two images.  Since we are generally not interested in sky and extreme left and right parts of the image, I then applied a mask to limit our image to a specific region of interest.  See image below.


![alt text][image2]



### Perspective transform

(The code for this function is in the Warping Function of the notebook)

Since our images were all the same size, I hard-coded values for the perspective transform.  A more robust solution would have to consider transforms for different-sized images, or standardize the size of the images before perspective transforming.

I chose the following source and destination points:

| Source        | Destination   |
|:-------------:|:-------------:|
| 200, 720      | 320, 720        |
| 1124, 720      | 920, 720      |
| 577, 456     | 320, 1      |
| 710, 456      | 920, 1        |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![alt text][image3]




###  Sliding Windows detection of Lane Lines

(Code for this step is in the various create_lane_line_windows functions in the notebook)

The lesson used a sliding window method in order to find lane lines.  This consisted of a histogram on various parts of the image in order to find the greatest grouping of pixels indicating lane lines.  In the pipeline for the video, I slightly altered the initial function and used create_lane_line_windows_alt1, which I find slightly more readable, though both functions are very similar.  I then essentially followed the steps in the lesson the rest of the code.  This led to a video which was about 98% successful in detecting the lines.  I ran into a couple tough spots which I tried several ways to improve.  Initially, I saved line fits and kept a running average.  I then compared the current fit to the average to try to find 'bad' fits.  This wasn't very successful, and ended up converging on one fit which didn't change after a point in the video.  I then tried keeping a running average of lane widths, which did better but still not ideal.  Finally, I tested the upper-most endpoint of the fit on each side and compared it to a running average.  I then threw out any that were over a threshold (50 pixels) and replaced the fit for that side.  This ended up performing better than the other methods, though still not flawlessly.  You can see an example of the sliding window below in the first image, and a polynomial fit in the second image.
I tested the other version of the function, create_lane_line_windows2, but found that didn't really improve the fits, and in some cases led to more bad fits.


![alt text][image4]
![alt text][image5]

### Curvature, Offset and Final Image

(Code found in cell 7 of the individual functions group and the process_video_image functions)

Most of this code comes straight from the lesson, with very little alteration.  I used the polynomial fit with the mathematical function and code from the lesson to calculate the curvature of the road and add it to the final image.  The vehicle offset from center was also calculated and information added to the image.

![alt text][image6]

---

## Output Video

#### Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

[Here's a link to the video output](project_video_output.mp4)

---

### Discussion

By far the most difficult part of this project was trying to correct the 2% of bad frames or fits in the video.  I spent many hours over several days trying to find a method which consistently could pick out bad frames.  I tried testing the fit coefficients, lane width, number of points detected, before finally deciding on a running average of the endpoints of the upper lane limbs as a standard (a margin around this point).  I also tried different thresholds and color maps in the color and gradient code, but ultimately didn't see much improvement to the current ones.  Although my final method is adequate, it probably still needs more improvement.  I didn't have time to test on the challenge videos, but I imagine changes could be made to improve detection especially in low-light or obscured conditions.  Not to mention the case where a road has no lines at all; this model would fail spectacularly.
For the second submission, I added an undistorted version of a test frame image, and changed the lane width correction to 600 pixels as mentioned in the first review.  The second sliding windows function was not implemented as it seemed to increase the number of bad fits in some cases.
