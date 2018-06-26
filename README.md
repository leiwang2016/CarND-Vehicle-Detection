**Vehicle Detection Project**

The goals / steps of this project are the following:

* Perform a Histogram of Oriented Gradients (HOG) feature extraction on a labeled training set of images and train a classifier Linear SVM classifier
* Optionally, you can also apply a color transform and append binned color features, as well as histograms of color, to your HOG feature vector.
* Note: for those first two steps don't forget to normalize your features and randomize a selection for training and testing.
* Implement a sliding-window technique and use your trained classifier to search for vehicles in images.
* Run your pipeline on a video stream (start with the test_video.mp4 and later implement on full project_video.mp4) and create a heat map of recurring detections frame by frame to reject outliers and follow detected vehicles.
* Estimate a bounding box for vehicles detected.

[//]: # (Image References)
[image1]: ./result/car_vs_nocar.png
[image2]: ./result/hog_example.png
[image3.1]: ./result/sliding_windows_1.png
[image3.2]: ./result/sliding_windows_2.png
[image3.3]: ./result/sliding_windows_3.png
[image3.4]: ./result/sliding_windows_4.png
[image4]: ./result/output_boxes.png
[image5]: ./result/bboxes_and_heat.png
[image6]: ./result/grey_output_boxes.png
[image7]: ./result/frame1258.jpg
[video1]: ./project_video_out.mp4

## [Rubric](https://review.udacity.com/#!/rubrics/513/view) Points
### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.

---
### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Vehicle-Detection/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.

You're reading it!

### Histogram of Oriented Gradients (HOG)

#### 1. Explain how (and identify where in your code) you extracted HOG features from the training images.

The code for this step is contained in the first code cell of the IPython notebook (or in lines # through # of the file called `some_file.py`).

I started by reading in all the `car` and `non-car` images.  Here is an example of one of each of the `vehicle` and `non-vehicle` classes:

![alt text][image1]

I then explored different color spaces and different `skimage.hog()` parameters (`orientations`, `pixels_per_cell`, and `cells_per_block`).  I grabbed random images from each of the two classes and displayed them to get a feel for what the `skimage.hog()` output looks like.

Here is an example using the `RGB` color space and HOG parameters of `orientations=9`, `pixels_per_cell=(8, 8)` and `cells_per_block=(2, 2)`:


![alt text][image2]

#### 2. Explain how you settled on your final choice of HOG parameters.

I tried various orientations. 12 orientations give more blurry outputs, and makes model overfit; although 9 orientations gives nice gradient outline, model doesn't get as good accuracy as 11 oreintations does.


#### 3. Describe how (and identify where in your code) you trained a classifier using your selected HOG features (and color features if you used them).

I built a classifier using HOG features (line 9). I compared a number of classifiers including logistic regression, linear SVM, random forest, gradient boosting machine. In the end
I chose to use linear SVM since it gives the best test accuracy and fastest training speed. Although GBM does outperform SVM in test accuracy, it tends to miss-classify non-vehicle regions as vehicle.

### Sliding Window Search

#### 1. Describe how (and identify where in your code) you implemented a sliding window search.  How did you decide what scales to search and how much to overlap windows?

I used varying sizes of sliding window from 1 to 3 scales, searching on the lower half of frames since this is more likely to contain cars.

![alt text][image3.1]
![alt text][image3.2]
![alt text][image3.3]
![alt text][image3.4]

#### 2. Show some examples of test images to demonstrate how your pipeline is working.  What did you do to optimize the performance of your classifier?

Ultimately I searched on two scales using LUV 3 channel HOG features plus spatially binned color and histograms of color in the feature vector, which provided a nice result, with 98% accuracy on test set.  Here are some example images:

![alt text][image4]

---

### Video Implementation

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (somewhat wobbly or unstable bounding boxes are ok as long as you are identifying the vehicles most of the time with minimal false positives.)
Here's a [link to my video result](./project_video_out.mp4)


#### 2. Describe how (and identify where in your code) you implemented some kind of filter for false positives and some method for combining overlapping bounding boxes.

I recorded the positions of positive detections in each frame of the video.  From the positive detections I created a heatmap and then thresholded that map to identify vehicle positions.  I then used `scipy.ndimage.measurements.label()` to identify individual blobs in the heatmap.  I then assumed each blob corresponded to a vehicle.  I constructed bounding boxes to cover the area of each blob detected.

Here's an example result showing the heatmap from a series of frames of video, the result of `scipy.ndimage.measurements.label()` and the bounding boxes then overlaid on the last frame of video:

![alt text][image5]

### Here is the output of `scipy.ndimage.measurements.label()` on the integrated heatmap from all six frames:
![alt text][image6]

### Here the resulting bounding boxes are drawn onto the last frame in the series:
![alt text][image7]

---

### Discussion

#### Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

In this project, I build classifier using HOG features. Using SVM classifier, I was able to get about 98% accuracy. However, looking at output videos, we can see that there is still some inaccuracies happening. For example, sometimes the classifier will label tree/road side as vehicle. Since both training and test set gives about similar accuracy, this is likely due to underfitting. In the future, I can try to add in color features in the classifier.

Another challenge I got building the classifier is the output bounding boxes sometimes only selects a part of the vehicle. So it doesn't give as accurate location information for the vehicle. This could be an issue when implementing the pipeline in real life self-driving car, where accurate location information is critical. Since the searching algorithm does not reward/penalize bounding box location accuracy, this is one big limitation in this algorithm. In the future, 
I would like to try implementing an algorithm with multi-dimensional output, including both classifier output and bounding box locations.

One additional limitation is the speed of inference. Since the algorithm involves searching for multiple bounding boxes sequentially for each frame, this makes it infeasible to do real-time self-driving. An algorithm that allows searching for multiple bounding boxes at the same time will help speed up the inference process.