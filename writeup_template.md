# Advanced Lane Finding Project

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

[image1]: ./examples/undistort_output.png "Undistorted"
[image2]: ./examples/undistort.png "Road Transformed"
[image3]: ./examples/binary_combo_example.png "Binary Example"
[image4]: ./examples/warped_straight_lines.png "Warp Example"
[image5]: ./examples/color_fit_lines.png "Fit Visual"
[image6]: ./examples/example_output.png "Output"
[equation1]: ./examples/equation1.png "equation1"
[equation2]: ./examples/equation2.png "equation2"
[equation3]: ./examples/equation3.png "equation3"
[equation4]: ./examples/equation4.png "equation4"
[video1]: ./project_output.mp4 "Video"

** [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points **

---
## Writeup / README

### Note: All code referenced here is in a IPython notebook located in "project.ipynb".

#### Camera Calibration

** 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image. **

The code for this step is contained in the first code cell named "Imports and Get Calibration Parameters Function".  

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `obj_points` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `img_points` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `obj_points` and `img_points` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function and applied this distortion correction to the test image using the `cv2.undistort()` function in the cell subsection "Calibrate Camera and Undistort Images" in the function `cal_undistort()`. The obtained result:

![alt text][image1]

#### Pipeline (single images)

**1. Provide an example of a distortion-corrected image.**

The procedure was the same as the camera calibration example displaying the original and the undistorted image side by side by using the `cal_undistort()` function.

![alt text][image2]

**2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.**

I used a combination of color and gradient thresholds to generate a binary image (thresholding steps are in the cell subsection "Pipeline Function" in the function `pipeline()`).  Here's an example of my output for this step.  (note: this is an image of a straight road from the test images)

![alt text][image3]

**3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.**

The code for my perspective transform includes a function called `warp()`, which takes as inputs an image (`img`), as well as source (`src`) and destination (`dst`) points.  I chose the hardcode the source and destination points in the following manner:

```
src = np.float32([(575,464),
                  (707,464),
                  (258,682),
                  (1049,682)])

ddst = np.float32([[240, 0],
                  [img_size[1] - 240, 0],
                  [240, img_size[0]],
                  [img_size[1] - 240, img_size[0]]])
```
This resulted in the following source and destination points:

| Source        | Destination   |
|:-------------:|:-------------:|
| 575, 464      | 240, 0        |
| 707, 464      | 480, 0        |
| 258, 682      | 240, 720      |
| 1049, 682     | 480, 720      |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![alt text][image4]

**4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?**

I implemented this step in the cell subsection "Implement Sliding Windows and Fit a Polynomial" in the function `sliding_windows()`, "Search For Lane-Line Pixels" in the function `line_search()`, and "Identify Lane-Line Pixels and Fit Their Positions With a Polynomial on a Rolling Average" in the function `poly_fit_average()`.
Then I did some other stuff and fit my lane lines with a 2nd order polynomial kinda like this:

![alt text][image5]

**5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.**

I implemented this step in the cell subsection "Measure lane Curvature and Offset" in the function `measure_curvature()`.

In the last section, I located the lane line pixels using their x and y pixel positions  to fit a second order polynomial curve with the equation below:

![alt text][equation1]

Radius of Curvature:

The radius of curvature ([awesome tutorial here]("http://www.intmath.com/applications-differentiation/8-radius-curvature.php")) at any point x of the function `x = f(y)` is given as follows:

![alt text][equation2]

In the case of the second order polynomial above, the first and second derivatives are:

![alt text][equation3]

So, our equation for radius of curvature becomes:

![alt text][equation4]

This is only the calculation of the radius of curvature based on pixels, now we have to convert the x and y from pixels space to meters. This is done by estimation the meters per pixel in both the x and y axis. Notice: In my code I go straight into meters, skipping pixels space.

Calculate offset from the center of the lane:

Assuming the camera is on the center of the car, we obtain get the center point of the undistorted image and the center of the lane from the calculated left and right x-axis fits. The offset is the difference between the center of the image/car and the center of the lane and multiplied by the x-axis  meters per pixel.

**6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.**



I implemented this step in the cell subsection "Draw Measurements Back down onto the Road" in the function `draw_lines()`.  Here is an example of my result on a test image:

![alt text][image6]

---

#### Pipeline (video)

**1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).**

Here's a [link to my video result](./project_output.mp4)

---

#### Discussion

**1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?**  

My only approach was simplicity and readability. I started with the code snippets from the quizzes and lesson. After I had the bare minimum down to test on the project video, I did an iterative process of manually optimizing each processing step at a time, starting with source and destination points and followed by pipeline, and sliding windows. The last step I did was to implement a smoothing technique that detected the lane lines over the last n frames of video, removing most of the wobbly lines in the project video test. My initial implementation was very long and complicated, after doing some research on better ways of doing it I stumbled upon a fellow classmate's blog and took his advice to improve my implementation (acknowledgment section). One of the challenges I faced was to get straight lane lines on the warped image from an image of a straight road as a reference to have accurate curvature and offset measurements and get good results from the project video test. I ended up using the best warped that resulted in the best performance on the project video test instead of the most straight lane lines on the warped image on an image from a straight road. My pipeline will fail in difficult terrain where there are multiple lines near the lane or in very dark shadows. To improve my pipeline, I have to implement a sanity check to remove outliers or bad detections and run the difficult frames through a modified sliding windows that can correctly detect the lane lines.

#### Acknowledgement

Radius of curvature:

http://www.intmath.com/applications-differentiation/8-radius-curvature.php

Smoothing technique:

http://davidaventimiglia.com/advanced_lane_lines.html
