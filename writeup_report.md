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
[video1]: ./project_video.mp4

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

Multi-scale sliding winodw approach mean the scale of the searching window would vary based on the distance of the other object relative to the us (i.e. vehicle), the closer the bigger searching window size should be. Starting in code cell named: Set Scale to 1.0, I started exploring different scales of searching window size and defining their search area in an image. Below are visualization of multi-scale sliding window search: 

![alt text][image4]
![alt text][image5]
![alt text][image6]
![alt text][image7]

Finally I combined all the sliding window scales into one pipeline, here is an example:

![alt text][image8]

#### 2. Show some examples of test images to demonstrate how your pipeline is working.  What did you do to optimize the performance of your classifier?

Ultimately I searched on two scales using YCrCb 3-channel HOG features plus spatially binned color and histograms of color in the feature vector, which provided a nice result.  Here are some example images:

![alt text][image4]
---

### Video Implementation

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (somewhat wobbly or unstable bounding boxes are ok as long as you are identifying the vehicles most of the time with minimal false positives.)
Here's a [link to my video result](./project_video.mp4)


#### 2. Describe how (and identify where in your code) you implemented some kind of filter for false positives and some method for combining overlapping bounding boxes.

I recorded the positions of positive detections in each frame of the video.  From the positive detections I created a heatmap and then thresholded that map to identify vehicle positions.  I then used `scipy.ndimage.measurements.label()` to identify individual blobs in the heatmap.  I then assumed each blob corresponded to a vehicle.  I constructed bounding boxes to cover the area of each blob detected.  

Here's an example result showing the heatmap from a series of frames of video, the result of `scipy.ndimage.measurements.label()` and the bounding boxes then overlaid on the last frame of video:

### Here are six frames and their corresponding heatmaps:

![alt text][image5]

### Here is the output of `scipy.ndimage.measurements.label()` on the integrated heatmap from all six frames:
![alt text][image6]

### Here the resulting bounding boxes are drawn onto the last frame in the series:
![alt text][image7]



---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

Here I'll talk about the approach I took, what techniques I used, what worked and why, where the pipeline might fail and how I might improve it if I were going to pursue this project further.  

