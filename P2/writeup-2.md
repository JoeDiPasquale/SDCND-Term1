
**Build a Traffic Sign Recognition Project**

The goals / steps of this project are the following:
* Load the data set (see below for links to the project data set)
* Explore, summarize and visualize the data set
* Design, train and test a model architecture
* Use the model to make predictions on new images
* Analyze the softmax probabilities of the new images
* Summarize the results with a written report


[//]: # (Image References)

[image1]: ./examples/visualization.jpg "Visualization"
[image2]: ./examples/grayscale.jpg "Grayscaling"
[image3]: ./examples/random_noise.jpg "Random Noise"
[]	: ./res/

###Data Set Summary & Exploration

####1. Provide a basic summary of the data set. 
I used the numpy library to calculate summary statistics:

* The size of training set is 34799
* The size of the validation set is 4410
* The size of test set is 12630
* The shape of a traffic sign image is (32,32,3)
* The number of unique classes/labels in the data set is 43

###Design and Test a Model Architecture

####1. Describe how you preprocessed the image data.
As a first step, I decided to convert the images to grayscale.
Then, I devided the data with 255, normalizing the image data to make searching faster.
Then I applied histogram equalization to extract more features.

####2. Describe what your final model architecture looks like including model type, layers, layer sizes, connectivity, etc.) 

My final model consisted of the following layers:

Layer         			Description	        					
------------------------------------------------------------------
 Input         			32x32x1 grayscale image   		
 Convolution1 5x5x6     	1x1 stride, valid padding, outputs 28x28x6 
 Max pooling	      		2x2 stride,  outputs 14x14x6 				
 Convolution2 5x5x16	    	1x1 stride, valid padding, outputs 10x10x16  	
 Max pooling	      		2x2 stride,  outputs 5x5x16 				
 Fully connected1		inputs 400, outputs 120	
 Droupout			keep probability 50%	
 Fully connected2		inputs 120, outputs 84	
 Fully connected3		inputs 84, outputs 43	
 Softmax			softmax cross entropy with logits			



####3. Describe how you trained your model.
To train the model, I used the training dataset with TensorFlow AdamOptimizer and the learning rate set to 0.001. I used batch size 256 and 30 epochs achieve 89% ~ accuracy on the validation data set.

####4. Describe the approach taken for finding a solution and getting the validation. 
My final model results were:
* validation set accuracy of 92.5% 


###Test a Model on New Images

####1. Choose five German traffic signs found on the web and provide them in the report. 
I've added signs in the attached 'res' folder.

####2. Discuss the model's predictions on these new traffic signs and compare the results to predicting on the test set. 

Here are the results of the prediction:

Image			        Prediction	        	
------------------------------------------------------------------
 Speed limit(20km/h)     Speed limit(20km/h) 				
 Speed limit(50km/h)     Speed limit(50km/h) 				
 Children crpssing	 Children crpssing										 
 Keep left		 Keep left      						
 Yield			 Yield	      		 


The model was able to correctly guess 5 of the 5 traffic signs, which gives an accuracy of 100%. 


