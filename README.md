
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

[image_calibration]: ./output_images/calibration.png "Calibration"

[image_undistorted]: ./output_images/undistorted.png "Undistorted"

[image_binary]: ./output_images/lane_5.png "Binary Exampl"

[perspective_transform_source]: ./output_images/perspective_transform_source.jpg "Perspective_transform"

[bird_view_5]: ./output_images/bird_view_5.png "Bird View"

[polyfit]: ./output_images/polyfit.png "Poly Fit"

[curvature]: ./output_images/curvature.png "Curvature Function"

[detection_1]: ./output_images/detection_1.png "Detetion 1"
[detection_2]: ./output_images/detection_2.png "Detetion 2"

[video1]: ./project_video_detected.mp4 "Video"

[shadow_example]: ./output_images/lane_10.png "Shadow Example"
[one_lane_example]: ./output_images/lane_11.png "One Lane Example"



## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points
###Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---
###Camera Calibration

The code for this step is contained in the first and second code cell of the IPython notebook located in "./main.ipynb" 

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

key step:
    ret, mtx, dist, rvecs, tvecs = cv2.calibrateCamera(objpoints, imgpoints, gray.shape[::-1], None, None)

![alt text][image_calibration]

###Pipeline (single images)

####1. Undistort the image

With the transform matrix calculated based on chessbord images, I undistored the images.

    undistorted = cv2.undistort(image, mtx, dist)

![alt text][image_undistorted]

####2. Lane detection

The lane detection are done with calculate the Gradient of the image. Both Magnitude of the Gradient and Direction of the Gradient are used.

In addition, I use light and hue to detect white and yellow lane.

I used a combination of color and gradient thresholds to generate a binary image.  Here's an example of my output for this step. 

![alt text][image_binary]

####3. Perspective transform

I performed a perspective transform of the binary image.

The code for my perspective transform appears in block 24 of `main.ipynb` 
From this image, I chose the the source points and hard code the destination points:
![][perspective_transform_source]

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 597, 448      | 264, 200      | 
| 683, 448      | 1041, 200     |
| 265, 678      | 265, 1200     |
| 1041, 678     | 1041, 1200    |

Get the transform matrix and its inverse matrix

    M = cv2.getPerspectiveTransform(src, dst)
    Minv = cv2.getPerspectiveTransform(dst, src)

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

    cv2.warpPerspective(binary, M, (1280, 1280), flags=cv2.INTER_LINEAR)

![alt text][bird_view_5]

####4. Polynomial fit

Then I detect the points using a "zone of interest" search method and fit the lane points with a 2nd order polynomial function:

To search on a new image, I use the "sliding windows" method. To search on a image based on a similar image (e.g. next frame in a video), the previous fit construct the "zone of interest"

Here is the result using both method:

![alt text][polyfit]

####5. Calculate the radius of curvature of the lane and the position of the vehicle

Base on the polynomial funtion, we can calculate the curvature of the line. 
![][curvature]
The current curvature of the vehicule is on the point y = Y_MAX
Supposing the camera is in the center of the vehicle, we can easily calculate the offset from center of the vehicle.

One point we need to pay attention is that when we want to present this in meter, we need to have the pixel-to-meter transform

    width_in_m = 1280 * xm_per_pix
    height_in_m = 1280 * ym_per_pix

I did this in "./main.ipynb", block 30 and 31

####6. Finally we provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in block 32 and 33 in `main.ipynb`.  Here is an example of my result on a test image:

![alt text][detection_1]
![alt text][detection_2]

---

###Pipeline (video)

####1. Final video output.  

Here's a [link to my video result](./project_video_detected.mp4)

---

###Discussion / To be improved

####1. The yellow line is no longer yellow when it is in shadow.

In this image:
![][shadow_example]

The yellow lane in the shadow is no loger yellow. With the paint tool of Windows, I find that the line color is blue so it cannot be detected by my yellow lane detector. Our eye think the color is yellow because we do automatical white balance in our brain.
I should do it also for line detection.

####2. Detect bad fit

I need a better algorithm to juge wether the fit is good (e.g. if two fit are parallel), take the confidence level (e.g. R^2) as a indication. If only one line is found with confidence, another algorithme should be designed for calculate the other line. Like the case in this picture: 
![][one_lane_example]


