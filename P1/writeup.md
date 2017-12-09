# **Finding Lane Lines on the Road** 

## Writeup Template

### You can use this file as a template for your writeup if you want to submit it as a markdown file. But feel free to use some other method and submit a pdf if you prefer.

---

**Finding Lane Lines on the Road**

The goals / steps of this project are the following:
* Make a pipeline that finds lane lines on the road
* Reflect on your work in a written report


[//]: # (Image References)

[image1]: ./examples/grayscale.jpg "Grayscale"

---

### Reflection

### 1. Describe your pipeline. As part of the description, explain how you modified the draw_lines() function.

The purpose of the pipeline is to compose several different operations together, apply them to an image, and produce an annotated image that shows where a lane on a road would be.

My pipeline consists of multiple steps:

Applying a color mask
Performing edge detection
Selecting regions to search for lane lines
Using the Hough transform to find line segments
Extrapolating the lane from the line segments provided by the Hough transform


Applying the color mask:
	Lanes on the road are typically found as a single color and designed to stand-out from the background. We can use this fact to help us filter out elements in the image that are irrelevant. To accomplish this, we use something called a color mask. A color mask can be applied to an image to remove all colors except for the ones we specify.

	This can be done using OpenCV’s cv2.inRange(src, lowerb, upperb[, dst]) → dst thresholding function. You provide the image and a range of the colors you want to create a mask for.

	For this project I created two masks: one for white lanes and one for yellow lanes.
	For the white lane I manually found values that seemed close and provided good results when run against the example images.

	The yellow lane was a bit more involved since it’s not as simple as setting all the channels to the same value. I used a color picker to grab the RGB components and plugged it into colorizer to transform it to HSV space. Using the cv2.cvtColor() function to convert the image from BGR colorspace to HSV, I was able to create a mask that isolated the yellow lane.

Performing edge detection:
	I then use the canny edge detector to pull out edges in the image. cv.Canny(image, edges, threshold1, threshold2, aperture_size=3)

Selecting regions to search for lane lines:
	
	After performing edge detection, there is still a fair amount of irrelevant edges that need to be ignored if we are to find the lane lines. We remove a majority of the image and focus on a region that we would most likely find lane lines.

Using the Hough transform to find line segments:
	
	The hough transform is the operation in the pipeline that finds line segments in the image and provides the most information of where the lanes lines could be. Initially I performed the hough transform on a single region which provided some inaccurate results. When I adjusted the pipeline to instead use two regions (one of the left line and one for the right), it significantly increased the accuracy.

	A lot of time was spent tweaking the parameters and manually tuning the hough transform to provide good results.

Extrapolating the lane from the line segments provided by the Hough transform:
	Once we have the line segments produced by the hough transform, we can find a line that would be suitable for annotating the initial image. I found cv.FitLine(points, dist_type, param, reps, aeps) to work relatively well at extrapolating the line from the lines found using the hough transform.





### 2. Identify potential shortcomings with your current pipeline


The challenge video exposed a couple flaws with my pipeline:

				Highly sensitive to color
				Requires hard coded regions
				Highly dependent on lane location


### 3. Suggest possible improvements to your pipeline

				Use information from past frames
				Better tuned hough transform and edge detection
				Automatically calculate region
				Automatically calculate color mask range