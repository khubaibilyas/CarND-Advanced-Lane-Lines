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

Example images of the pipeline are saved to output_images folder. Running the last block of `main.ipynb` will execute the pipeline for each test image and print all intermediate images using matplotlib.

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the first code cell of the IPython notebook located in "./examples/example.ipynb" (or in lines # through # of the file called `some_file.py`).  

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![test1 raw][test_images/test1.jpg]
![test1 undistorted][output_images/test1_undistorted.jpg]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

Image in each frame of the video is undistorted using `cv2.undistort()` before further processing.
![Sample undistorted Image][output_images/test3_undistorted.jpg]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used a combination of color and gradient thresholds to generate a binary image. This step is done in the function `performBinaryThreshold` in ./examples/main.ipynb. 

Step 1 : Extract saturation image from BGR image by using BGR2HLS transform.

Step 2 : Find absolute sobel_x and sobel_y and scale them to uint8.

Step 3 : Compute magnitude of the sobel by calculating `np.sqrt(scaledImgX**2 + scaledImgY**2)`

Step 4 : Compute the combined binary threshold image by applying a threshold for both saturation and sobel magnitude. The following line in 
`performBinaryThreshold` sets combined image pixels to 255 if both saturation image and sobel-magnitude image are above a certain threhsold: 
                `combined_img[np.logical_and(satImg > 150, magImg > 70)] = 255`

![binary thresholded image.][output_images/test1_thresholded_binary.jpg]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

Perspective transform is done in the function `getwarpedImg` which takes the image, source and destination points and returns the transformed image. Source and destination points are set arbitrarily in the pipeline function : 

Source Points : 
top_right = (725, 455)
top_left = (555, 455)
botom_right = (1280, 680)
botom_left = (0, 680)

Destination Points : 
top_right = (0, Image height)
top_left = (0, 0)
botom_right = (Image width, 0)
botom_left = (Image width, Image height)

![Perspective transformed image][output_images/test2_perspective_transformed.jpg]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

Identifying lane lines from bird's eye view is done in the function `fitPolyToLaneLines`. The algo in this function can be explained in the following steps : 

Step 1 : Histogram of the warped image and lane line validity is computed in the function `getLaneIdxAndValidity`. Lane validity is a flag to indicate if lane lines have enough pixels to be considered as one. If this is False, previous polynomial fit is used to identify the corresponding lane.

Step 2 : Set sliding windows of a determined height and margin from start to end for each lane line. Position of each window is determined by the maximum of sum of pixels underneath it.

Step 3 : Extract non zero pixels of the warped image underneath all the windows seperately for each lane line.

Step 4 : Fit polynomial of degree 2 for these non zero pixels and store them to a global.

Step 5 : Scale the polynomial coefficients using a constant `alpha` to be closer to the polynomial coefficients of previous frame. This is done in the function `scalePolyFit` : Current_polynomial = Previous_polynomial + alpha * (Current_polynomial - Previous_polynomial). 
This is done to reduce flickering and minimise the effects of drastic change in polynomial coefficients.

Step 6 : Color the region between the the lane lines. This is done in `getLaneColoredImage`. 


#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

I did this in the function `findCurvatureAndVehPos`. Radius of curvature and Position of vehicle with respect to centre are both globals.

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

The image with regions between the lane lines colored is warped using `getwarpedImg` but with inverse source and destination points. This is done in the pipeline function and the resulting image is overlapped with the source image. The output looks like this : 

![test3_final_overlapped][output_images/test3_final_overlapped.jpg]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./project_video_output.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

I noticed that my thresholded binary image is not perfect and has a lot of noise. Perhaps more fine tuning of this can make it more robust.  
