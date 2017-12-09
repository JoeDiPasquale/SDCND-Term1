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

[image1]: ./examples/undistort_output.png "Undistorted"
[image2]: ./test_images/test1.jpg "Road Transformed"
[image3]: ./examples/binary_combo_example.jpg "Binary Example"
[image4]: ./examples/warped_straight_lines.png "Warp Example"
[image5]: ./examples/example_output.png "Output"


## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points
###Camera Calibration

####1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the second code cell of the IPython notebook located in "P4.ipynb' of the root folder

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect 17 out of 20 chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![alt text][image1]

###Pipeline (single images)

####1. Provide an example of a distortion-corrected image.
To demonstrate this step, I will describe how I apply the distortion correction to one of the test images like this one:
![alt text][image2]
####2. Describe how (and identify where in your code) The following functions were implemented to perform Histogram normalization, binary Sobel transformation (including magnitude and direction), red color channel and saturation channel. 

Unfortunately, the optimal combination of these filters which is able to separate pixels of lane line from background on snapshots from all three videos was not found. Shadows and glares are quite challenging. (For first two videos such filter combination was successfully found).

So, an adaptive threshold was applied (see description below) on a red channel image for white line finding and on a linear combination of the red and saturation channels - for the yellow line.

![alt text][image3]

####3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

Points for image warping were selected so parallel lines becomes really parallel. It creates so-called Bird's Eye View. We also crop images to skip areas with hood and sky. An example is provided below.

```
src = np.float32(
	[[0, 673], 
	[1207, 673], 
	[0, 450], 
	[1280, 450]])
    
dst = np.float32(
	[[569, 223],
	 [711, 223],
	  [0, 0], 
	  [1280, 0]])

```


I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![alt text][image4]

####4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?


An equidistant is needed in case only one line is well determined (the other one is the equidistant because lane lines are parallel). It is known that an equidistant for a polynomial function is a higher order polynomial function. However, it is needed not higher then 3rd order polynomial in order to make line fitting robust and stable.  That is why we use an approximation: we create a list of equidistant points for the given polinomial (points laying on a straight lines, perpendicular to the given polynomial at selected points on a desired distance) and fit them with a same order polinomial function.The `np.polyfit` function is used for fitting.

One of the key ideas of the project is the usage of the reasonable minimal order of polinomial functions for lines fitting. The function `best_pol_ord` chooses such order. It starts from a linear function and, if it does not perform well enough increases the polinomial order (up to 3). It returns polinomial coefficients and mean squared error of the selected approximation.
The function stops increasing the order if it does not help (mean squared error drops not significant in case of higher order) or in case the mean squared error is small enough (`< DEV_POL`).

It is possible to use a simple low pass filter or an alpha-beta filter for smoothing polynomial coefficients of the same degree. But for two polynomial with different orders a dedicated function `smooth_dif_ord` was introduced. It calculates average x position of points for a given function f(y)  at y values from the new dataset and fit these new average points with polinomial of desired degree.



####5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

Radius of lanes curvature was calculated out of polinomial lane lines approximation by a formula from a (http://www.intmath.com/applications-differentiation/8-radius-curvature.php), which was recomended in the course notes. If the calculated radius is bigger than `MAX_RADIUS` then the `MAX_RADIUS` value is returned because such a big radiuses of curvature are very unaccurate and can be considered as a straight line. Offset from the lane center was estimated as lane ofset in pixels, converted to meters.

####6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

The 'draw_lane()' code in the P4 can draw found lines on images and perform inverse perspective transformation with *Minv* matrix. It is realized in the same way as it was shown in the course lectures. It also inprints radius of curvature of the road and an  offset from the lane center. 

`get_lane` function just calls find function to detect lane on an image and visualize results

![alt text][image5]

---

###Pipeline (video)

####1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).
Below are the links to all three video outputs.
[challenge_video_proc](./video_proc/challenge_video_proc.mp4)

[project_video_proc](./video_proc/project_video_proc.mp4)

[harder_challenge_video_proc](./video_proc/harder_challenge_video_proc.mp4)

---

###Discussion

####1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?
Problem 1: Noise interfering with detection of lane lines, resulting in lines with higher curvature being drawn
           Solutions:
	           *  increase the minimum threshold to filter noice
	           *  add an offset so that parts that are too right are not included in the histogram.
Problem 2: No lane line detected (usually right lane line)
           Solution:
            Relax x gradient and S channel thresholds using a while loop that relaxes the thresholds by a tiny amount and then repeats the detection process if no lane line is detected. This allows us te relax the thresholds when no lane line is detected without adding noise to frames where lane lines were detected on the first go (e.g. if we'd just changed the thresholds directly).
