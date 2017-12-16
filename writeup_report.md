## Writeup Report
### Writeup report of Vehicle Deteciton And Tracking Project
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
[image1]: ./ExampleOutputImages/TrainingDataVisualization.JPG
[image2]: ./ExampleOutputImages/VisualizeHOG.JPG
[image3]: ./ExampleOutputImages/OneWindowSearch.JPG
[image4]: ./ExampleOutputImages/Scale10.JPG
[image5]: ./ExampleOutputImages/Scale15.JPG
[image6]: ./ExampleOutputImages/Scale20.JPG
[image7]: ./ExampleOutputImages/Scale30.JPG
[image8]: ./ExampleOutputImages/FinalSildingWindowSearch.JPG
[image9]: ./ExampleOutputImages/HeatMap.JPG
[image10]: ./ExampleOutputImages/TresholdHeatMap.JPG
[image11]: ./ExampleOutputImages/SciPy.JPG
[image12]: ./ExampleOutputImages/FinalBounding.JPG
[image13]: ./ExampleOutputImages/FinalTestImages.JPG
[video1]: ./project_video_out.mp4


## [Rubric](https://review.udacity.com/#!/rubrics/513/view) Points
### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---
### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Vehicle-Detection/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading the writeup. Please also see the IPython notebook - "Vehicle-Detection-And-Tracking.ipynb" which has the detailed annotation I put along with the code.

### Histogram of Oriented Gradients (HOG)

#### 1. Explain how (and identify where in your code) you extracted HOG features from the training images.

Before extracing HOG features, I started by reading in all the training images- `vehicle` and `non-vehicle` images.  Here are some examples of the `vehicle` and `non-vehicle` classes:

![alt text][image1]

The code of extracting HOG features from image is contained in the code cell named: Convert Image to Histogram of Oriented Gradients (HOG) in the ipython notebook. I defined a method named get_hog_features to do this job. Here are some examples of visualizing the comparison between original images and their HOG features shown images. 

![alt text][image2]

I then explored different color spaces and different `skimage.hog()` parameters (`orientations`, `pixels_per_cell`, and `cells_per_block`).  I grabbed random images from each of the two classes and displayed them to get a feel for what the `skimage.hog()` output looks like.


#### 2. Explain how you settled on your final choice of HOG parameters.

I tried various combinations of parameters and finally found that, with colorspace = YCrCb, orientation=9, pixels per cell = 8, cells per block = 2, HOG = ALL, Feature extracion time < 80 secs,  I could have the classifier that will be trained later to reach 98.76% test accuracy.  

#### 3. Describe how (and identify where in your code) you trained a classifier using your selected HOG features (and color features if you used them).

The code of training the classifier is contained in the code cell named: Train The Classifier. I trained a linear Support Vector Machine (SVM) using default classifier parameters and the training data I had processed with the features that combined HOG, bin spatial and color hist. The trained was able to achieve a test accuracy of 98.76%.

### Sliding Window Search

#### 1. Describe how (and identify where in your code) you implemented a sliding window search.  How did you decide what scales to search and how much to overlap windows?

As suggested in the lesson, it is beneficial to create a multi-scale sliding window approach. But before that, I have the function finds_car which is the function to search the vehicle using only one scale, a full window size. Here is an example of the result: 

![alt text][image3]

Multi-scale sliding winodw approach mean the scale of the searching window would vary based on the distance of the other object relative to the us (i.e. vehicle), the closer the bigger searching window size should be. Starting in code cell named: Set Scale to 1.0, I started exploring different scales of searching window size and defining their search area in an image. Below is visualization of final multi-scale sliding window search with different scales: 

Scale 1.0:

![alt text][image4]

Scale 1.5:

![alt text][image5]

Scale 2.0:

![alt text][image6]

Scale 3.0:

![alt text][image7]

To speed things up, I only extract HOG features just once for the entire region of interest (i.e. lower half of the image) and subsample that array for each sliding window. Overlap is decided by how many cell distance I move from step to step. Each searching window has 8 x 8 cells and I have cells_per_step = 2, so it result in a search window overlap of 75%. 

Finally I combined all the sliding window scales into one pipeline, here is an example:

![alt text][image8]

The images now are showing all the bounding boxes for where my classifier reported positive detections, then I build a heat-map from these detections in order to combine overlapping detections and later use threshold to remove false positives.

To create a heat map, I added "heat" (+=1) for all pixels within windows where a positive detection that reported by my classifier. Here is an example of heat map from the final sliding window example I showed above: 

![alt text][image9]

In order to reject areas affected by false positives, a threshold =2 is created, it would result the thresholded heat map as below. As you can see the false poitive on the left has been rejected.

![alt text][image10]

Once obtaining the thresholded heat-map from a list of bounding boxes, use the label() function from scipy.ndimage.measurements. to figure out how many cars in each frame and which pixels belong to which car. Here is an example of result: 

![alt text][image11]

Then I took the labeled image and put bounding boxes around the labeled regions. 

![alt text][image12]

#### 2. Show some examples of test images to demonstrate how your pipeline is working.  What did you do to optimize the performance of your classifier?

Ultimately I searched on two scales using YCrCb 3-channel HOG features plus spatially binned color and histograms of color in the feature vector, which provided a nice result.  I ran the pipeline on all the test images, here are some example images:

![alt text][image13]
---

### Video Implementation

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (somewhat wobbly or unstable bounding boxes are ok as long as you are identifying the vehicles most of the time with minimal false positives.)
Here's a [link to my video result](./project_video_out.mp4)


#### 2. Describe how (and identify where in your code) you implemented some kind of filter for false positives and some method for combining overlapping bounding boxes.

Derived from the heat map method I mentioned above, in code cell named: "Define a Class to Store Data from Vehicle Detections", I created a Vehicle_Detect class for storing the detection rectangles of previous n frames. This allows me to create the heat map not from single frame but from the past n frames. And the threshold I defined it as at least half of past n frames have to return positive detection. This definetely enhance the robustness of the pipeline when processing continuous frames from the video.

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

I found the the experimentation of finding what features works the best took me quite long hours of time. I ended up using YCrCb 3-channel HOG plus spatially binned color and histograms of color but thinking there is still more room I could improve.

I also found the pipeline with the thresholded heat map is generated based on signle frame does not work well, as I tried that pipeline on the test short video, I found the bounding boxes are quite wobboling. As hinted in the lesson, turns out I need to consider previous frames, which is kind of the concept of tracking, in order to increase the robustness of the detection.  

As other computer vision based techniques, enviromental concerns, like lighting condition, weather, are always the major obstacles to the technique that heavily rely on images. Also the classifier may also fail detecting vehicle if the specific vehicle is not like/represented in the training set we fed the classifier during the training. 

To make the vehicle detectiong and tracking pipeline more robust, we may want to consider the detected vehicle's speed, it will enhance the prediction of its next location on the image. I personally consider that, rather than rely only on images processed technique, the vehicle detection and tracking definetely needs sensor fusions, i.e. combine the inputs from other sensors, like radar and lidar, in order to elimate more false poitives or even worse false negatives. I think this definetely explain why long range/short range radars are still being considered as major inputs of object detection. I consider camera images more suitable for traffic sign/light classifier, or even to identify pedestrian (along with lidar data or ultrasonic sensor data).
