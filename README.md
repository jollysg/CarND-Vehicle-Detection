**Vehicle Detection Project**

The goals / steps of this project are the following:

* Perform a Histogram of Oriented Gradients (HOG) feature extraction on a labeled training set of images and train a classifier Linear SVM classifier
* Optionally, you can also apply a color transform and append binned color features, as well as histograms of color, to your HOG feature vector. 
* Note: for those first two steps don't forget to normalize your features and randomize a selection for training and testing.
* Implement a sliding-window technique and use your trained classifier to search for vehicles in images.
* Run your pipeline on a video stream (start with the test_video.mp4 and later implement on full project_video.mp4) and create a heat map of recurring detections frame by frame to reject outliers and follow detected vehicles.
* Estimate a bounding box for vehicles detected.

[//]: # (Image References)
[image1]: ./output_images/sampleImages.png
[image2]: ./output_images/hog_image1.png
[image3]: ./output_images/detectionsample1.png
[image4]: ./output_images/detectionsample2.png
[image5]: ./examples/bboxes_and_heat.png
[image6]: ./examples/labels_map.png
[image7]: ./examples/output_bboxes.png
[video1]: ./output_images/project_video_out8.mp4
[video2]: ./output_images/test_video_out8.mp4

## [Rubric](https://review.udacity.com/#!/rubrics/513/view) Points

###Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

###Histogram of Oriented Gradients (HOG)

####1. Explain how (and identify where in your code) you extracted HOG features from the training images.

The code for this step is contained in lines 93 through 103 of extract_features method in the file called `vehicleDetection.py`. hog method from skimage library was used to identify the features. The usage is in the get_hog_features method in the same file.    

I started by reading in all the `vehicle` and `non-vehicle` images.  Here are a few examples of each of the `vehicle` and `non-vehicle` classes:

![alt text][image1]

Here is an example of an `RGB image` and its corresponding image with HOG parameters of `orientations=9`, `pixels_per_cell=(8, 8)` and `cells_per_block=(2, 2)`, after conversion to `YCrCb` color space.

![alt text][image2]

####2. Explain how you settled on your final choice of HOG parameters.

I then explored different color spaces and different `skimage.hog()` parameters (`orientations`, `pixels_per_cell`, and `cells_per_block`).  I grabbed random images from each of the two classes and displayed them to get a feel for what the `skimage.hog()` output looks like. After trying various parameters, I settled on using the 'YCrCb' color space with HOG parameters of 9 orientations, 8 pixels per cell and 2 cells per block. I also ended up using all the channels in the HOG.


####3. Describe how (and identify where in your code) you trained a classifier using your selected HOG features (and color features if you used them).

A Linear SVC classifier was used in this project with the help of sklearn library. The code for this can be seen in the lines 389 through 453 of the vehicleDetection.py file. The training was done using the vehicle and non-vehicle images provided with the project. For features, apart from HOG features, histogram (bin size 16) and spatial (16 by 16) features were also used for training. All the features were fed into the SVC model for training using a single dimension vector of 6108 features. With the parameters chosen, the test accuracy of the classifier was more than 99%. For a detailed list of parameters, please refer to lines 401 through 411 in the vehicleDetection.py

###Sliding Window Search

####1. Describe how (and identify where in your code) you implemented a sliding window search.  How did you decide what scales to search and how much to overlap windows?

I started searching the lower half of the image (yPositions 360 to 720) with a window size of 96 by 96 at first. But that lead to a poor performance of the detector, with a lot of missed detections, especially when the vehicles were nearer in the frame. In many cases, the detector would only detect vehicles partially. So I decided to use multi-scale windows. The window sizes chosen were 64, 96, 128, 192 and 224 with 75% overlap to identify both vehicles close as well as far away.

####2. Show some examples of test images to demonstrate how your pipeline is working.  What did you do to optimize the performance of your classifier?

Ultimately I searched using YCrCb 3-channel HOG features plus spatially binned color and histograms of color in the feature vector, which provided a nice result.  Here are some examples of the detections, heat maps and the labels:

![alt text][image3]

![alt text][image4]

---

### Video Implementation

####1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (somewhat wobbly or unstable bounding boxes are ok as long as you are identifying the vehicles most of the time with minimal false positives.)

Following are the links to my video results:

[Project Video][video1]

[Test Video][video2]


####2. Describe how (and identify where in your code) you implemented some kind of filter for false positives and some method for combining overlapping bounding boxes.

I recorded the positions of positive detections in each frame of the video.  From the positive detections I created a heatmap and then thresholded that map to identify vehicle positions.  I then used `scipy.ndimage.measurements.label()` to identify individual blobs in the heatmap.  I then assumed each blob corresponded to a vehicle.  I constructed bounding boxes to cover the area of each blob detected.  

This worked fine for the individual frame, however it led to very wobbly boxes, also occasional dissappearance of them. So to further refine the detections, I implemented a short term memory of the bounding boxes in the code. All the detections for the last 15 frames were stored, and a combined heat map was generated from all of them. To reduce the false negatives, thresholding was used, with the value chosen experimentally to half the number of frames in history. This resulted in a relatively less wobbly boxes in the video. The code for this can be found in the lines 580 to 586 in the process_video function of the vehicleDetection.py file.
  
---

###Discussion

####1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

The approach taken has been elaborated in the implementation section of the writeup. following are the problems that I see with the implementation:


#####1. Performance around the edges of the region of interest
For performance optimization, the window search is carried out in a region of interest of every frame, which is mainly bottom half of the image. The detector performs relatively well when the car is well within the region of the interest, however on the edges, the boxes become wobbly. As a result, when the car goes too far away from the camera or towards the edge(the white car in the project video), it is not detected. For this window scaling will have to be optimized to cover all the pixels in the corner properly. Currently due to the scales chosen, the pixels on the right size of the image are not searched. 

#####2. Performance when the road condition changes
There is a portion of the video where the color of the road changes from dark gray to light due to change in the conditions (tar to concrete). The detections fail around this point. For this perhaps the classifier needs to be trained with a few images from this segment of the road.

#####3. Multiple cars in proximity to each other
The detector will fail when multiple cars are in proximity of each other in the image. Due to the implementation, the detector will group the heatmaps from both the cars together and there is no way for it to know that the maps are for different objects. For this the detected number of objects will have to be tracked separately with a more sophisticated filter like a Kalman filter.


   

