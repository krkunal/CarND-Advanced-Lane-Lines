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

[image1]: ./output_images/distortion_correction.jpg "Distortion correction"
[image2]: ./output_images/Undistorted_example.jpg "Undistorted Example"
[image3]: ./output_images/test2_binary.jpg "Binary Example"
[image4]: ./output_images/perspective_transform.jpg "Perspective Transformed"
[image5]: ./output_images/test5_lane_polylines.jpg "Lane polyline Example"
[image6]: ./output_images/test1_final.jpg "Output1"
[image7]: ./output_images/test2_final.jpg "Output2"
[image8]: ./output_images/test3_final.jpg "Output3"
[image9]: ./output_images/test4_final.jpg "Output4"
[image10]: ./output_images/test5_final.jpg "Output5"
[image11]: ./output_images/test6_final.jpg "Output6"
[image12]: ./output_images/straight_lines1_final.jpg "Output7"
[image13]: ./output_images/straight_lines2_final.jpg "Output8"
[video1]: ./test_videos_output/project_video.mp4 "Video"


### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the 2nd code cell of the IPython notebook adv_lane_lines.ipynb".

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  The Calibration matrix & Distortion coefficient found from the object & image points were persisted. I applied this distortion correction to one of the calibration images using the `cv2.undistort()` function and obtained this result: 

![alt text][image1]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

Using the Calibration matrix & distortion coefficient, I undistorted a test image test5.jpg and the distortion corrected image is like this:
![alt text][image2]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used a combination of color and gradient thresholds to generate a binary image (thresholding steps in cell no. 4 of the notebook). Sobel X gradient thresholding was applied on the L channel of the input image in HLS format, followed by a threshold on the S-channel of the image. The binary image was further masked using a region of interest focussing on a triangular region in the bottom half of the image. Then, I applied a Gaussina Blur with kernel size = 3 to smoothen out the pixels. Here's an example of my output for this step on image test2.jpg.

![alt text][image3]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes functions called `get_perspective_matrices()` & `xform_perspective()`, which appear in the 7th code cell of the notebook.  The `get_perspective_matrices()` function takes as inputs an image (`img`), as well as source (`src`) and destination (`dst`) points.  I've wrote functions `find_perspective_xform_pts()` & `process_image()` to automatically find the source points from an image in the 5th code cell of the notebook. I used the source points found using these functions and chose the destination points based on judgement.

This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 281, 695      | 400, 700      | 
| 1104, 695     | 978, 700      |
| 836, 530      | 978, 300      |
| 495, 530      | 400, 300      |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![alt text][image4]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

The function `find_lane_pixels()` in the 9th code cell of the notebook uses the histogram + sliding window approach to find lane pixels from the warped binary images. I used the hyperparameters - `no. of windows = 9`, `margin = 50 pixels`, and `minimum no. of pixels = 30` for the sliding window implementation. Then, in the `fit_polynomial()` function, I've used np.polyfit() to fit 2nd order polynomials on the lane pixels found by sliding window method. I've also done look ahead filter implementation in the function `search_around_poly()` in code cell 11. This function uses the polynomial coefficients found in the previous frame to search for lane pixels in the next frame. The output of the polynomial fitting on the image test5.jpg is:

![alt text][image5]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

I calculated the radius of curvature form the polynomial coefficients using the Radius of curvature formula applied at the bottom of the image. Then I took the minimum of the two radii and called that the ROC of the lane. 
For the Offset calculation, I found the midpoint of the lane from the identified lane pixels at the bottom of the image and subtracted that from the midpoint of the image. Both these numbers were converted from the pixel space to world space to get the offset in meters. 
The radius of curvature & offest calculations are in the function `measure_curvature_real()` in the code cell 12 of the notebook.

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

The end to end pipeline is implemented in the function `run_image_pipeline()` in code cell 13. All the test image outputs are in the outout_images directory. Here is an example of my result on a test image:

![alt text][image6]
![alt text][image7]
![alt text][image8]
![alt text][image9]
![alt text][image10]
![alt text][image11]
![alt text][image12]
![alt text][image13]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

I've used Line class to preserve the state of variables of interest across frames, then applied sanity check on the identified lane lines. If the sanity check fails, then the values until previous frame will be used, otherwise the values of current frame are considered for the lane highlighting. I've applied look ahead filter for more efficient identification of lane lines. Also, I've taken average of last 5 frames for the x locations of the left & right lanes to smoothen the positioning of lane lines.   
Here's a [link to my video result][video1]

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

Apart from the basic components like Distortion correction, Color/Gradient thresholding, Perspective transformation, and Sliding window approach for lane pixel identification, I've implemented Look ahead filter and Lane class for preserving the variables of interest across the video frames. I've used sanity checks on lane width & lane lines' parallel check to decide when to fall back on values found until previous frame. To smoothen out the lane lines, I've taken the average of the x values at the bottom of the image for last 5 good runs.

___What didn't work & what to improve___ - 
* I tried using a sanity check on radii of curvature for left and right lanes being approximately same. But, the radii of curvatures were differing a lot in some cases and, seemingly, were not a reliable measure. 
* Sometimes, the function for identifying lane pixels itself fails with an exception. This is happening a lot in challenge videos in the frames with low light. This failure can be handled by falling back on values found in previous frames like I did in the sanity check failure scenario. 
* The Color/Gradient thresholding is pretty basic as of now and it doesn't seem to do the job in challenge videos. It can be improved further to be able to handle sharp gradients on the road & both low light & normal light conditions in the frames.