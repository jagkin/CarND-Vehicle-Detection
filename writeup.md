--

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
[image2]: ./examples/heatmap_and_boundingboxes.png
[video1]: ./project_video.mp4

## [Rubric](https://review.udacity.com/#!/rubrics/513/view) Points
### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---
### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Vehicle-Detection/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!

### Histogram of Oriented Gradients (HOG)

#### 1. Explain how (and identify where in your code) you extracted HOG features from the training images.

The code for this step is contained in the cells under section "Feature extraction" in the IPython notebook.
method extract_features in class VehicleDetectionTrainer() has the code to extract HOG features from a list of files.

I started by reading in all the `vehicle` and `non-vehicle` images.  Here is an example of one of each of the `vehicle` and `non-vehicle` classes:

![alt text][image1]

I did not consider data augmentation (to balance the dataset for car and notcar images) as the dataset is already somewhat balanced.

I initially implemented the functions to extract color features (spatial bin and color histogram). I realized that color features help increase the accuracy by a coupe of %age points but also increase the feature vector length (and the training/prediction time) considerably. I decided to use HOG alone as feature to save the time time/memory.


#### 2. Explain how you settled on your final choice of HOG parameters.

I then explored different color spaces and different `skimage.hog()` parameters (`orientations`, `pixels_per_cell`, and `cells_per_block`). I prepared a table of paramsets with different color spaces (RGB, YUV, LUV, HSV) and permutations of orient, pix_per_cell and cell_per_block and calculated the feature extraction time, training time, prediction time, accuracy for each config set.

The final choice of the HOG params is based on the reasonabe accuracy (> 97%) and short feature length (and hence lower feature extraction, training and prediction time).

Here are the final params used.

    # Features
    self.color_space = 'LUV' # Can be RGB, HSV, LUV, HLS, YUV, YCrCb
    self.orient = 12  # HOG orientations
    self.pix_per_cell = 16 # HOG pixels per cell
    self.cell_per_block = 2 # HOG cells per block
    self.hog_channel = 'ALL' # Can be 0, 1, 2, or "ALL"


#### 3. Describe how (and identify where in your code) you trained a classifier using your selected HOG features.

I decided to use LinearSVC as classifier function based on the suggestion in the lesson videos.
I did not get a chance to try out other classifier functions for this project.

The classifier function training is implemented as method train_classifier() in class VehicleDetectionTrainer().

I initially extracted color features and used StandardScaler() to normalize the features. But in the interest of quicker training/prediction I chose to drop the use of color features and Scaler.

### Sliding Window Search

#### 1. Describe how (and identify where in your code) you implemented a sliding window search.  How did you decide what scales to search and how much to overlap windows?

The sliding window search is implemented as methods find_cars() and process_image() in the notebook.
find_cars() is copied from the lesson exercises.
process_image() calls find_cars() with varying y_start_stop and scale values as suggested by Drew in the lesson video.
The y_start_stop and scale values were tuned based on trial and error method but follow the logic that the cars closer to the camera are bigger in the image and get smaller as they move away from the camera (towards the horizon).

#### 2. Show some examples of test images to demonstrate how your pipeline is working.  What did you do to optimize the performance of your classifier?

Code cell "Test the pipeline with a sample image" in the notebook shows result of the pipeline on a sample test image.

![alt text][image2]
---

### Video Implementation

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (somewhat wobbly or unstable bounding boxes are ok as long as you are identifying the vehicles most of the time with minimal false positives.)
Here's a [link to my video result](./marked_project_video.mp4)


#### 2. Describe how (and identify where in your code) you implemented some kind of filter for false positives and some method for combining overlapping bounding boxes.

I recorded the positions of positive detections in each frame of the video.  From the positive detections I created a heatmap and then thresholded that map to identify vehicle positions.  I then used `scipy.ndimage.measurements.label()` to identify individual blobs in the heatmap.  I then assumed each blob corresponded to a vehicle.  I constructed bounding boxes to cover the area of each blob detected. 

In case of video, I recorded the bounding boxes detected for last 2 seconds (= 50 frames at 25fps) and used them to compute the heat map over time to minimize the temporary false positives.

In the test video, the vehicle is always driving on the left most lane of the road so I applied a x_start_stop = [250, 1280] to ignore the road structures on the left side of the lane.

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

Problems/Issues faced:
1. I ran in to some issues at the beginning due to mixing up to cv2.imread and mpimg.imread for reading the images.

Likely failure scenarios:
1. The pipeline is likely to fail in case of road with sharp curves as the window sizes/scales were tuned for the project video.
2. Video pipeline is likely to fail in case of video where car is driving on lanes other than left most lane (as I have set the x_start to 250).
3. The pipeline may fail when there is big change in lighting (eg. Shadows, road surface changing the color etc)

Possible improvements:
1. Additional datasets (eg. Udacity labellled data) could be used to improve the classifier
2. Data augmentation (Applying lighting/perspective transformation on the existing images) can be used to create additional data for classifier training. 
3. The lane detection algorithm from previous project can be repurposed to detect the road edges and then limit the search window (for vehicles).
