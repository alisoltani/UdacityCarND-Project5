
**Vehicle Detection Project**

The goals / steps of this project are the following:

* Perform a Histogram of Oriented Gradients (HOG) feature extraction on a labeled training set of images and train a classifier Linear SVM classifier
* Optionally, you can also apply a color transform and append binned color features, as well as histograms of color, to your HOG feature vector. 
* Note: for those first two steps don't forget to normalize your features and randomize a selection for training and testing.
* Implement a sliding-window technique and use your trained classifier to search for vehicles in images.
* Run your pipeline on a video stream (start with the test_video.mp4 and later implement on full project_video.mp4) and create a heat map of recurring detections frame by frame to reject outliers and follow detected vehicles.
* Estimate a bounding box for vehicles detected.

[//]: # (Image References)
[image1]: ./output_images/car_not_car.jpg
[image2]: ./output_images/hog0.jpg
[image3]: ./output_images/windows.jpg
[image4]: ./output_images/vehicle_detection_example1.jpg
[image5]: ./output_images/vehicle_detection_example2.jpg
[image6]: ./output_images/vehicle_detection_example3.jpg
[image7]: ./output_images/detection_example1.jpg
[image8]: ./output_images/detection_example2.jpg
[image9]: ./output_images/detection_example3.jpg
[video1]: ./video_output.mp4

## [Rubric](https://review.udacity.com/#!/rubrics/513/view) Points
###Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---
### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Vehicle-Detection/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!

### Histogram of Oriented Gradients (HOG)

#### 1. Explain how (and identify where in your code) you extracted HOG features from the training images.

The code for this step is contained in the 7th code cell of the IPython notebook (and in lines 25 through 42 of the file called `vehicledetection.py`).  

I started by reading in all the `vehicle` and `non-vehicle` images.  Here is an example of one of each of the `vehicle` and `non-vehicle` classes:

![alt text][image1]

I then explored different color spaces and different `skimage.hog()` parameters (`orientations`, `pixels_per_cell`, and `cells_per_block`).  I grabbed random images from each of the two classes and displayed them to get a feel for what the `skimage.hog()` output looks like.

Here is an example using the `YCrCb` color space and HOG parameters of `orientations=18`, `pixels_per_cell=(8, 8)` and `cells_per_block=(2, 2)`:


![alt text][image2]

#### 2. Explain how you settled on your final choice of HOG parameters.

I tried various combinations of parameters and different colorspaces, and basically went through a trial and error phase to look at both output result and how long it took. 

First I started with BGR(RGB) channels, but the performance was not statisfactory for use in hog. I tried with the L channel in HLS, which seemed to work adequetly, but the best performance was obtained with Y channel in YCrCb (although I finally added all three channels because I was losing track of the car sometimes). I tried with orientation 9 and 18 , with 9 I had a 0.975 accuracy and with 18 a 0.99 so I stuck with 18. Using values larger than 2 cells per block did not help.

It is important to remember to normalize the feature so one feature does not dominate. This is done by the StandardScalar (line 293).

#### 3. Describe how (and identify where in your code) you trained a classifier using your selected HOG features (and color features if you used them).

I trained a linear SVM using a linear svc and hog features, histogram bin size of 16 bins and spatial bins of 32x32. The training was done in the 9th cell in the ipynb (and in lines 303-317 of `vehicledetection.py`), and achieves accuracy of around 0.991. There are 13704 feature total.

### Sliding Window Search

#### 1. Describe how (and identify where in your code) you implemented a sliding window search.  How did you decide what scales to search and how much to overlap windows?

The windows consist of smaller windows of 64x64 with a lot of overlap (0.85 in both x and y direction), a slightly bigger window of 92x92 and 0.8 overlap, and finally a 128x128 window with a 0.75 overlap. These windows are arranged in a matter such that the searching is done relative to car size, i.e. if the car is close and to the right we don't search for it in the 64x64 windows because those are for identifying cars that are further ahead.

I also experimented with larger windows, but this combination seemed to give best results.

![alt text][image3]

#### 2. Show some examples of test images to demonstrate how your pipeline is working.  What did you do to optimize the performance of your classifier?

Ultimately I searched on two scales using YCrCb 3-channel HOG features plus spatially binned color and histograms of color in the feature vector, which provided a nice result.  Here are some example images:

![alt text][image4]
![alt text][image5]
![alt text][image6]
---

### Video Implementation

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (somewhat wobbly or unstable bounding boxes are ok as long as you are identifying the vehicles most of the time with minimal false positives.)
Here's a [link to my video result](./video_output.mp4)


#### 2. Describe how (and identify where in your code) you implemented some kind of filter for false positives and some method for combining overlapping bounding boxes.

I recorded the positions of positive detections in each frame of the video.  From the positive detections I created a heatmap and then thresholded that map to identify vehicle positions.  I then used `scipy.ndimage.measurements.label()` to identify individual blobs in the heatmap.  I then assumed each blob corresponded to a vehicle.  I constructed bounding boxes to cover the area of each blob detected. I then saved the heatmap and bounding boxes for consecutive frames to use in instances where there are shadows and changes in the road.   

Here's an example result showing the heatmap from a series of frames of video

![alt text][image7]

![alt text][image8]

![alt text][image9]

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

One issue that was faces was the issue of balancing false positives vs losing vehicle tracking. If the parameters were set too tight, false positives would dissapear but then I would lose tracking of the cars in some places. This required a lot of parameter tuning to get right with my current algorithm.

The implementation I have also takes a long time to execute and will not be able to run effectivly in real-time. This issue needs to be addressed further, but for the purpose of this project I think it is good enough as is. A better approach that would track better and be simpler to execute is to, instead of recording previous bounding boxes, to find the centroid of each box, use a Kalman filter to track them from frame to next frame and then only search around previously found vehicles and windows were new cars may appear (along the horizon or from the left right).

Further due to the way I made the windows, it can only identify vehicles on the right. Any oncomming vehicle will not be in the field of view, or if there are hills/valleys where the vehicles do not fit into the y axis range chosen. This should also be taken into account for further development.

Finally, vehicle detection from this project should be combined with lane tracking from the previous project.
