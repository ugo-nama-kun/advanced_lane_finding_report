##Writeup Template
###You can use this file as a template for your writeup if you want to submit it as a markdown file, but feel free to use some other method and submit a pdf if you prefer.

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

[image0]: ./my_results/calibration_points.png "Example of Detected Points in the Image"
[image1]: ./my_results/camera_calib.png "Original and Undistorted Images"
[image2]: ./my_results/road_undistorted.png "Original and Undistorted Images"
[image3_0]: ./my_results/test6.jpg "Original (not undistorted)"
[image3]: ./my_results/value_image.png "Value Example"
[image3_2]: ./my_results/binary_image.png "Binary Example"
[image4]: ./my_results/warped.png "Warp Example"
[image5]: ./my_results/fit_poly.png "Fit Visual"
[image6]: ./my_results/lane_show.png "Output"
[video1]: ./project_video.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points
###Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---
###Writeup / README

####1. Provide a Writeup / README that includes all the rubric points and how you addressed each one. 

This file.

###Camera Calibration

####1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the third and fourth code cells of the IPython notebook located in "advanced_lane_finding.ipynb".

I followed the standard camera calibration procedure as presented in the lecture of thi project. I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0 (2D plane), such that the object points are the same for each calibration image.  

Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection. In order to do this process, I converted the RGB image to gray scale, and then input into 'cv2.findChessboardCorners' function. The example of the result of this process is shown as below:

![alt text][image0]

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![alt text][image1]

###Pipeline (single images)

####1. Provide an example of a distortion-corrected image.
Using the obtained parameters of camera calibration, I corrected the distortion using the `cv2.undistort()` function (13-th cell). 
![alt text][image2]
####2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.
The code is provided in the 28-th cell (the function `get_threshold_binary_image()`). I converted the RGB image to the HSV color space, and then I used "Value" image from HSV to extract the line information. The example of value image is shown as below:

![alt text][image3_0]
![alt text][image3]

In order to extract lines, I used the binarized-value image with threshold (`hsv_select()`), sobel filter with respact to the x cordinate (`gradx = abs_sobel_thresh()`) and the directionan filter (`dir_threshold()`). And finally extracted the image pixels in the region of interest (`region_of_interest()`). I made many possible other filtering method (like `yellow_select()`, canny filter, ROI, etc.), but their combination was not strainght forward. Therefore, I used just a few of them. Here's an example of my output for this step. 

![alt text][image3_2]

####3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The perspective transform is presented in the 25-th cell of my ipython notebook. The `transform_image()` function takes as inputs an image (`img`), as well as source (`src`) and destination (`dst`) points.  I chose the hardcode the source and destination points in the following manner:

```
d0 = 430
d1 = 57
d2 = 100
d3 = 300
src = np.float32([[img_size[0]/2 - d0, img_size[1]],
                  [img_size[0]/2 - d1, img_size[1]/2 + d2],
                  [img_size[0]/2 + d1, img_size[1]/2 + d2],
                  [img_size[0]/2 + d0, img_size[1]]])
dst = np.float32([[img_size[0]/2 - d3, img_size[1]],
                  [img_size[0]/2 - d3, 0],
                  [img_size[0]/2 + d3, 0],
                  [img_size[0]/2 + d3, img_size[1]]])

```

This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 210, 720      | 290, 720        | 
| 583, 460      | 290, 0      |
| 697, 460     | 990, 0      |
| 1070, 720     | 990, 720        |

I verified that my perspective transform was working as expected by showing the transformed images to verify that the road lines appear parallel in the warped image.

![alt text][image4]

####4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

I almost entirely followed the method that was presented in the project lecture. The `line_finder()` function is the main process of the line identification process (111-th cell in my ipython notebook). Undistorted images are binalized by the method as described in the 2nd section, and then the perspective transform convert the image to the top-view perspective image. The blind-search is described in the 111-th cell (`line_finder()` function). The function that utilize the previous result of the line finder is described in the `find_next_lines()` function in 114-th cell of mt ipython notebook. 

I took the sliding window approach as presented in the lecture (initialliy I tried to use the convolution approach, but the result was worse and then I turned to the sliding window approach. This process took time for me and then my submission of my project work was delayed as a result. This experience was great because I could learn that it is really hard to decide when to switch the method fundamentally). 

And I fit my lane lines with a 2nd order polynomial kinda like this (yellow lines):

![alt text][image5]

####5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

I did this process in the 125-th cell of my ipython notebook (the `get_curvature()` function). I fllowed the method presented in the lecture to obtain the curvature of the left and right lines. I used the simple mean of the left and right line curvatures to show in the result image. 

The position of the vehicle was extracted by using the right and left bottom (nearest) lane points detected by using the method described in the previous section. I got the center point (lane-center) of these two points and compared with the center of the (undistorted) image. I simply subtracted the horizontal element of the image-center from the lane-center. If this difference is positive, the vehicle souhld be in the left-side of the lane. This process is described in the `get_curvature()` function too.

####6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

The entire process is packed at the 158-th cell of my ipython notebook. The `LineFinder` class handles the line finding method and the previous result of the line fitting. This is my result:

![alt text][image6]

---

###Pipeline (video)

####1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

This is the [link to my video result](./project_result.mp4)

---

###Discussion

####1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

During this project, I felt that the way to find the best filtering method was very problematic. It was very ad-hoc and many parameters, hard to tune, and its evaluation is not numeric but the visual confirmation. I felt that some machine-learning related evaluation measure is required in this filtering process. 

Tried to optimize the binary image to clearly extract the line images, but I still feel that this is not perfect. We can see that the line fitting sometimes fails when in the "white road" area. I feel that I may be able to remove this phenomenon by detecting the obtained points as anomary in the "time-stream" of the detected points. 