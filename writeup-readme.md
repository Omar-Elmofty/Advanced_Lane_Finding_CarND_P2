### Writeup / README
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

[image1]: ./writeup-readme_images/calibration3.jpg
[image2]: ./writeup-readme_images/calibration3_calibrated.jpg
[image3]: ./writeup-readme_images/thresholded_img.jpg
[image4]: ./writeup-readme_images/warped_img.jpg
[image5]: ./writeup-readme_images/lane_highlighed.jpg
[image6]: ./writeup-readme_images/lane_unwarped.jpg
[video1]: ./project_video.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points


### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

To calibrate camera the following is done the calibrate_camera() function:
Step1: Initiate object points,this is a grid that represent the chess board corners of the images that will be used to calibrate the camera.
Step2: Calibration images are loaded 
Step3: For each of the training images, convert image to gray scale, then find the coordinates for the chess board corners using the function  cv2.findChessboardCorners. The object points and the image points are then appended to the objectpoints and imagepoints lists respectively.
Step4: cv2.calibrateCamera uses objectpoints and imagepoints lists to produce the camera matrix and the distortion coefficients.

The function undistort_image uses the camera matrix and distortion coefficients (output from camera_calibration function) to undistort images.

An example of an image before and after calibration is shown below.
![Before Calibration][image1] 
![After Calibration][image2]



### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

Here is an example of an undistorted image of the road.
![Undistorted_image_of_road][image6]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

The following functions used color gradients and color transforms to threshold the image:

First function: mag_thresh(gray, sobel_kernel=3, mag_thresh=(0, 255))
This is a function that thresholds the image based on the strength of the magnitude of the gradient of the input grayscale image. First the gradient in the x and y directions were computed using Sobel function, then the magnitude is computed mag = sqrt(gradx^2+grady^2). The magnitude is then thresholded to produce a binary image.

Second function: dir_thresh(gray, sobel_kernel=3, thresh=(0, np.pi/2)):
This function computes the direction of the gradient by computing the arctangent of the gradient in y over the gradient in x. The direction is then thresholded and a binary image is produced. Again the input to this function is a grayscale image.


Third fucntion: hls_select(img, l_thresh=0,s_thresh=0):
This function thresholds the image based on the l and s color channels. First the image is converted to hls color space using the function cv2.cvtColor, then the image is thresholded and binarized based on the strength of the s and l channels. 

An example of the thresholded image (both color and gradient combined) is shown below.

![alt text][image3]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The below 2 functions uses perpective transforms:

First function: perspective_transform(img,verticies):
This function takes in a normal road image and applies a perspective transform to get a birds eye view of the road. This function takes in the verticies which represent the corners of the lane in the normal road images, then applies the transform to warp the image and have the corners of the image warped to the following  destination vertices. 
[[0,720],[0,0],[1280,0],[1280,720]]

Second function: transform_back(img,verticies):
This function essentially reverses what the function perspective_transform(img,verticies) does. The function transforms the birds eye view image of the road back to the real perspective view of the road. The function takes in the verticies of the destination coordinates of the lane onto the transformed image.

Both of these functions are used in the pipeline(img, camera_calib) to transform the road images as required. The source vertices used to locate the corner of the lane are [[110,700],[550,460],[750,460],[1270,700]].

Here is an example of the transformed image (from perspective to birds eye view):

![alt text][image4]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

To identify lane lines, a Line class was defined.
In the line class the function fit_polynomial(self, binary_warped,nwindows,margin,minpix) was used to first find the lane line pixels using the sliding window approach. Then second order polynomial is fitted into the found pixels using np.polyfit function.

The lane lines were initiated and fitted in the pipeline function as shown:

#Detect Left lane
left_lane_region = left_lane.lane_region(binary_warped,150)
left_lane.find_x_base(left_lane_region,'left')
left_lane.fit_polynomial(left_lane_region,9,100,50)
left_lane.calc_lane_curvature()
left_lane.check_sanity_and_update((0,640),50)

#Detect right lane
right_lane_region = right_lane.lane_region(binary_warped,150)
right_lane.find_x_base(right_lane_region,'right')
right_lane.fit_polynomial(right_lane_region,9,100,50)
right_lane.calc_lane_curvature()
right_lane.check_sanity_and_update((640,1280),50)


#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

The radius of curvature was calculated using function calc_lane_curvature() which is part of the line class. This function was called in the pipline function as shown below:

#Detect Left lane
left_lane_region = left_lane.lane_region(binary_warped,150)
left_lane.find_x_base(left_lane_region,'left')
left_lane.fit_polynomial(left_lane_region,9,100,50)
left_lane.calc_lane_curvature()
left_lane.check_sanity_and_update((0,640),50)

#Detect right lane
right_lane_region = right_lane.lane_region(binary_warped,150)
right_lane.find_x_base(right_lane_region,'right')
right_lane.fit_polynomial(right_lane_region,9,100,50)
right_lane.calc_lane_curvature()
right_lane.check_sanity_and_update((640,1280),50)

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

See below image with lane lines highlighted clearly. This is the ouput of the function pipeline(img, camera_calib).

![alt text][image5]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./output_images/project_video_output.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

So far my pipeline has been tested on only the project's video, which was taken during daytime. The pipeline was not tested on any nightime videos. Most of my strugles were in finding the right color channels to use as well as defining the thresholds for the color channels and the gradients. Therefore, I believe that if the lighting conditons change(for example from day to night), then the pipline might fail since the thresholds are only tuned to the conditions in the project video.

An improvement to the pipeline would be to add more thresholding conditions that would work for other lighting condtions. These new threholds will have to be determined by trial and error.

Since color or gradient threholding is very mechanical and can be very time consuming to tune the thresholds to work perfecty, another technique for detecting the lane pixels might be prefereable. Convelutional neural networks would be a much better technique that will provide good results.

