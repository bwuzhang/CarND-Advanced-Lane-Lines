
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

[image1]: ./camera_cal/calibration1.jpg "Distorted"
[image2]: ./output_images/calibration1.jpg "Undistorted"
[image3]: ./output_images/straight_lines1_2.jpg "Undistorted_example"
[image4]: ./output_images/straight_lines1_3.jpg
[image5]: ./output_images/straight_lines1_4.jpg
[image6]: ./output_images/straight_lines1_5.jpg
[image7]: ./output_images/straight_lines1_6.jpg
[image8]: ./output_images/straight_lines1_7.jpg
[image9]: ./output_images/test1_3_red_threshed.jpg
[image10]: ./output_images/test1_3_V_threshed.jpg
[image11]: ./output_images/test1_3_white_1.jpg
[image12]: ./output_images/test1_3_white_2.jpg
[image13]: ./output_images/test1_3_white_3.jpg
[image14]: ./output_images/test1_3_yellow.jpg
[image15]: ./output_images/test1_2.jpg
[image16]: ./output_images/test1_3.jpg
[video1]: ./project_video.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the first and second part of the IPython notebook located in "./project.ipynb".

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result:

![alt text][image1]

![alt text][image2]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

To demonstrate this step, I will describe how I apply the distortion correction to one of the test images like this one:
![alt text][image3]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used a combination of color thresholds to generate a binary image (part 3).  Here's an example of my output for each step.

Original image
![alt text][image15]
Red channel thresholding
![alt text][image9]
V channel thresholding
![alt text][image10]
Yellow thresholding
![alt text][image11]
White thresholding from BGR
![alt text][image12]
White thresholding from HSV
![alt text][image13]
White thresholding from HSL
![alt text][image14]
All combined
![alt text][image15]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform is in part 4.
The `src` and `dst` are hardcoded from one of the `straight_lines2.jpg` such that the length of each dash lane segment is approximately 1.1 times of the width of the lane. This can be verified by looking at Google Map Satellite images.

This resulted in the following source and destination points:

| Source        | Destination   |
|:-------------:|:-------------:|
| 262, 700      | 550, 710        |
| 429, 579      | 550, 600      |
| 874, 579     | 650, 600      |
| 1060, 699      | 650, 710        |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![alt text][image5]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

I used the following steps to find lane pixels:
1. Generate histogram of lane pixels to initialize two starting points at the bottom of the image.
2. Split the images into 9  windows vertically.
3. For each window, from the previous window center, with a margin of 10 pixels, find the new window center.
4. Repeat step 3 for all remaining windows.
5. If the result of last frame exists, use that as the initialization of each window center and a margin of 30 to locate new centers.

![alt text][image6]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

I did this in lines #89 through #91 in part 4.
I use the equation from the course to calculate the curvature of both lanes, and take the average. Since the vertical pixel in the bird-view lane image represents same length in real world as the horizontal pixel, I don't need to apply any scaling factor to x or y axis.

The position is calculated by measure how many pixels the middle of the two lanes are away from the center (pixel #600), and then convert the pixel value to actual length in real world.
#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I overlay the lane image on the original image in part 6 add the text in part 7.

![alt text][image8]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./project_video.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

The pipeline was not stable at the beginning. Although it successfully detected lanes in the most times, it failed when the road texture changed. I fixed this by introducing a tracking feature for the video result generation. In each frame, the new lane window center is restricted to at most 30 pixel away from the last frame by making the assumption that lane doesn't have very sharp curves. It works well on `project_video.mp4`
