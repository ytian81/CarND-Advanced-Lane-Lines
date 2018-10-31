## Advanced Lane Finding Project

[//]: # "Image References"

[image0]: ./camera_cal/calibration2.jpg "Original"
[image1]: ./output_images/undistort_images/calibration2.jpg "Undistorted"
[image2]: ./output_images/undistort_images/straight_lines1.jpg "Road Transformed"
[image3]: ./output_images/gradient_threshold/sobel_combined.jpg "Binary Example"
[image31]: ./output_images/color_threshold/color_combined.jpg "Binary Example"
[image4]: ./output_images/perspective_transform/curve_lane_transformed.jpg "Warp Example"
[image5]: ./output_images/lane_line_pixel_finding/histogram.jpg "Fit Visual"
[image51]: ./output_images/lane_line_pixel_finding/priori.jpg "Fit Visual"
[image6]: ./output_images/final/curved_lane_anotated.jpg "Output"
[video1]: ./output_images/videos/project_video.mp4

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code is under _Camera calibration_ section in the notebook. I use checkerboard calibration images to compute camera matrix and distortion coefficients. More specifically, I need to use `cv2.findChessboardCorners` function to find checkerboard corners in image as `imgpoints`. Then I manuall hard coded the `objpoints` as (x, y, z) on a plane of `z=0`. x has value of 0-8. y has value of 0-5. `cv2.calibrateCamera` function is used to compute the camera matrix and distortion coefficients, which are used by `cv2.undistort` to undistort images. The following are a straight lane lines images and its undistorted version. To see the differece, you might need to open both pictures and compare side by side.

![alt text][image0] ![alt text][image1]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

To demonstrate this step, I will describe how I apply the distortion correction to one of the test images like this one:
![alt text][image2]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I use gradient thresholds to get a binary output image shown below. The code is under _Threshold binary image_ section. Specifically, I do threshold on x direction gradient, total magnitude and gradient direction. Y direction gradient is not desired because it introduces extra noise. The combination logic is `sobel_combined[(sobel_x == 1) | ((sobel_total == 1) & (sobel_dir == 1))] = 1`. Bascially either x direction gradient exceeds threshold or both total gradient exceed threshold and gradient direction is within certain range.

![alt text][image3]

I also use color thresholds on Red channel and Saturation channel shown below. The red channel is generally good to detect white and yellow lines while saturation channel is robust to shadow.

![alt text][image31]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

I use the image of _straight_lines1.jpg_ to hard code source point and destination point of perspective transform. The code is at _Perspective transform_ section. 

This resulted in the following source and destination points:

| Source        | Destination   |
|:-------------:|:-------------:|
| 205, 720     | 320,720     |
| 590, 453     | 320,0       |
| 690, 453     | 960,0       |
| 1115,720     | 960,720     |

I verify that implementation by taking a look at trasnformed result of curved lane. They are still parallem in new perspective.

![alt text][image4]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

I use histogram peaks and sliding window to figure lane-line pixels. After first pass, I can use the last frame result as a prior to search for lane-line pixels, unless the number of pixels is significantly small. The code is under _Detect lane pixels_ section.

![alt text][image5]
![alt text][image51]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

I use the formula in the course to calculate the curvature and use distance between lane center and image center to determine lateral offset of vehicle. The code is under _Curvature and vehicle lat offset calculation_ section

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I use the code provided in the course to put everything back to orignal image. The code is under _## Transform back to original image_ section

![alt text][image6]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result][video1]. If you cannot open it, it is `./output_images/videos/project_video.mp4`

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

Apart from techinques taught and used above, I also use a little trick to smooth result. It is a memory of tracked lane line pixels. Each tick, I will fit a second order curve to the whole memory pixels. The memory also has a limited size. Whenever it is overflowed, discard the oldest data. I tried my pipeline on both challenage video and harder_challege video. It doesn't work well. The challange one has a mixing edge in the center of a line, which I believe makes lane detection fail. The other one is basically break the assumption of flat ground, so the perspective transform will generally fail. I also notice that using the perspective transform, we can get rid of masking technique used in project 1. 



To further improve this project, online perspective transform may be needed. We don't want hard code perspective transform matrix, because it can fail easily on unenven ground. To make the result smoother, we could consider give different weights to different frame fitting result, instead of treating them equally.