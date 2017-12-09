##Writeup Template
###You can use this file as a template for your writeup if you want to submit it as a markdown file, but feel free to use some other method and submit a pdf if you prefer.

---

**Vehicle Detection Project**

The goals / steps of this project are the following:

* Perform a Histogram of Oriented Gradients (HOG) feature extraction on a labeled training set of images and train a classifier Linear SVM classifier
* Optionally, you can also apply a color transform and append binned color features, as well as histograms of color, to your HOG feature vector. 
* Note: for those first two steps don't forget to normalize your features and randomize a selection for training and testing.
* Implement a sliding-window technique and use your trained classifier to search for vehicles in images.
* Run your pipeline on a video stream (start with the test_video.mp4 and later implement on full project_video.mp4) and create a heat map of recurring detections frame by frame to reject outliers and follow detected vehicles.
* Estimate a bounding box for vehicles detected.

[//]: # (Image References)
[image1]: ./examples/car_not_car.png
[image2]: ./examples/HOG_example.jpg
[image3]: ./examples/sliding_windows.jpg
[image4]: ./examples/sliding_window.jpg
[image5]: ./examples/bboxes_and_heat.png
[image6]: ./examples/labels_map.png
[image7]: ./examples/output_bboxes.png
[video1]: ./project_video.mp4

## [Rubric](https://review.udacity.com/#!/rubrics/513/view) Points
###Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---
###Writeup / README

####1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Vehicle-Detection/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!



### I. Histogram of Oriented Gradients (HOG)

#### 1. Extract HOG features from the training images.

1. Read all vehicle and non-vehicle images.

2. Use `skimage.feature.hog(training_image, [parameters=parameter_values])` to extract the HOG features and HOG visualisation.
    * Wrapped in function `get_hog_features`.
    


#### 2. Choosing HOG parameters.

* I wanted to optimize for HOG parameters systematically, so I wrote a script `hog_experiment.py` that enables me to easily run through different HOG parameters and save the classifier accuracy, the HOG visualization (image) and bounding boxes overlaid on the video frame (image).
    * I later used `p5-for-tuning-classifier-parameters.ipynb`.
    * I used a spreadsheet to note down the parameters used each time, the accuracy, training time and the quality of the bounding boxes for each of the six test images. (E.g. was each car detected? If so, how well was it covered by the bounding boxes (how many bounding boxes + did it cover the car in full or only partially?) How many false positives were there?)
* I then picked the HOG parameters based on classifier accuracies and looking at the output images. I would like to make this process more rigorous instead of basing it on intuition. 
    * It was difficult to do this because the classifier accuracy was usually above 99% and was often shown as 1.0 even if the classifier later drew many false positive bounding boxes.
    * I later realized that the accuracy had been so high because I'd only been using 500 images from each category (vehicles and non-vehicles). But I still couldn't rely only on the classifier accuracy as a measure because a higher accuracy didn't always give me 'better' bounding boxes.



#### 3. Train a classifier using selected HOG features and colour features.

1. Format features using `np.vstack` and `StandardScaler()`.
2. Split data into shuffled training and test sets
3. Train linear SVM using `sklearn.svm.LinearSVC()`.


### II. Sliding Window Search

#### 1. Implement a sliding window search.

1. Define windows to search using helper function `slide_window`.
    * Restricted search space to lower half of the image (altered value of variable `y_start_stop`) because cars only appear on the road and not in the sky.
2.  Implement sliding window search using helper function `search_windows`.
    * For each window, 
        * extract features for that window, 
        * scale extracted features to be fed to the classifier, 
        * predict whether the window contains a car using our trained Linear SVM classifier, 
        * and save the window if the classifier predicts there is a car in that window.


#### 2. Show some examples of test images to demonstrate how your pipeline is working.  What did you do to try to minimize false positives and reliably detect cars?

* Measures to reliably detect cars in single images: I restricted the search space to only the lower portion of the image, i.e. the non-sky and mostly non-tree portion.

Sample image:

![Sample image of bounding boxes around classified windows](./readme_images/1.2.png)
---

### III. Video Implementation

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (somewhat wobbly or unstable bounding boxes are ok as long as you are identifying the vehicles most of the time with minimal false positives.)

[Video result](project_video_proc.mp4)

#### 2. Describe how (and identify where in your code) you implemented some kind of filter for false positives and some method for combining overlapping bounding boxes.

Pipeline:
1. For each frame, apply the image pipeline and add the detected bounding boxes (positive detections) to a global list `bboxes_list`.
2. Construct a heatmap from the most recent 20 frames of video (or using the number of frames available if there have been fewer than 20 frames before the current frame).
 
3. Reject false positives: threshold the heatmap.
4. Draw bounding boxes around the area of each labelled area detected.

Sample image of heatmap:
![](./readme_images/heatmap.png)

Sample image of labels:
![](./readme_images/labels.png)

---

## 4. Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

Initial problems:

1. When building the video pipeline, my functions failed to detect any vehicles for one of the frames. This returned `AttributeError: 'NoneType' object has no attribute 'shape' `. 
    * It turned out there was no return object for my function.
2. My classifier accuracy was 1.0 for a long time. It turned out I'd only been using the first 500 images from each category (vehicle and non-vehicle).

Persistent problems:

1. I can't assess the strength of the parameter combination using only one instance of bounding boxes detected in an image. The bounding boxes detected by the same combination of parameters varies.
2. The pipeline still misses out cars often. It is especially likely to fail when there are shadows.
3. Interestingly the model performed much better when I didn't use spatial or histogram features. This shows that adding more features (even if they sound like they might help) can make a model do worse.
