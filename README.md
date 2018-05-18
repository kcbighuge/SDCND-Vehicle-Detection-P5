# Vehicle Detection and Tracking
[![Udacity - Self-Driving Car NanoDegree](https://s3.amazonaws.com/udacity-sdc/github/shield-carnd.svg)](http://www.udacity.com/drive)


The goals / steps of this project are the following:

* Perform a Histogram of Oriented Gradients (HOG) feature extraction on a labeled training set of images and train a classifier Linear SVM classifier
* Implement a sliding-window technique and use a trained classifier to search for vehicles in images.
* Run the pipeline on a video stream and create a heat map of recurring detections frame by frame to reject outliers and follow detected vehicles.
* Estimate a bounding box for vehicles detected.

Here are links to the labeled data for [vehicle](https://s3.amazonaws.com/udacity-sdc/Vehicle_Tracking/vehicles.zip) and [non-vehicle](https://s3.amazonaws.com/udacity-sdc/Vehicle_Tracking/non-vehicles.zip) examples to train the classifier.  
- These example images come from a combination of the [GTI vehicle image database](http://www.gti.ssr.upm.es/data/Vehicle_database.html), the [KITTI vision benchmark suite](http://www.cvlibs.net/datasets/kitti/), and examples extracted from the project video itself.   
- You are welcome and encouraged to take advantage of the recently released [Udacity labeled dataset](https://github.com/udacity/self-driving-car/tree/master/annotations) to augment the training data.  

---


[//]: # (Image References)
[image1]: ./examples/car_notcar.png
[image2]: ./examples/HOG_car_notcar.png
[image3]: ./examples/slide_subwindows.png
[image4]: ./examples/boxes_test_imgs.png
[image5]: ./examples/heat_bboxes.png
[video1]: ./output_video.mp4


## Implementation    
Below are the steps taken to implement the pipeline.  

### Histogram of Oriented Gradients (HOG)

#### 1. Extract HOG features from the training images.

The code for this step is contained in code cell #18 of the IPython notebook `VehicleDetection_v1.ipynb`.

I started by reading in all the `vehicle` and `non-vehicle` images.  Here is an example of one of each of the `vehicle` and `non-vehicle` classes:

![alt text][image1]

I then explored different color spaces and different `skimage.hog()` parameters (`orientations`, `pixels_per_cell`, and `cells_per_block`).  I grabbed random images from each of the two classes and displayed them to get a feel for what the `skimage.hog()` output looks like.

Here is an example using the `YCrCb` color space and HOG parameters of `orientations=8`, `pixels_per_cell=(8, 8)` and `cells_per_block=(2, 2)`:


![alt text][image2]

#### 2. Final choice of HOG parameters.

I tried various combinations of parameters and settled on the following settings as a good balance of speed performance and classificaion accuracy.
```
color_space = 'YUV'     # Can be RGB, HSV, LUV, HLS, YUV, YCrCb
orient = 11             # HOG orientations
pix_per_cell = 16       # HOG pixels per cell
cell_per_block = 2      # HOG cells per block
hog_channel = 'ALL'     # 0, 1, 2, or "ALL"
```

#### 3. Train a classifier using HOG features and color features.

I trained both a linear SVM and Multilayer Pereceptron (MLP) using normalized HOG features and spatial features (size `(16,16)`) in code cells #19-20.

Here is their accuracy on the test set...
```
6.5 Seconds to train SVC...
Test Accuracy of SVC =  0.9933

12.2 Seconds to train MLP...
Test Accuracy of MLP =  0.9978
```


### Sliding Window Search

#### 1. Implement a sliding window search.  

I decided to search 2 different window scales with 75% overlap by subsampling the single whole image HOG extraction in code cells #24-25.

![alt text][image3]

#### 2. Examples of test images to demonstrate the pipeline is working.  

The final classifier was optimized by searching on 2 window scales using YUV 3-channel HOG features plus spatially binned color in the feature vector.  Here are some example images:

![alt text][image4]


---

### Video Implementation

#### 1. Final video output.  
Here's the [output video file](./output_video.mp4).

And here's a [youtube link](https://youtu.be/XHXD3tRlTyM) to the [output file combined with lane finding](./lane_plus_vehicle_video.mp4).

#### 2. Implement a filter for false positives and combining overlapping bounding boxes.

A filter for false positives is implemented in code cells #29-32.
1. I recorded the positions of positive detections in each frame of the video.  
2. From the positive detections I created a heatmap and added it to a queue of the past 11 video frames. This combined heatmap is then thresholded to identify vehicle positions.  
3. I used `scipy.ndimage.measurements.label()` to identify individual blobs in the heatmap, assuming each blob corresponded to a vehicle.  
4. Finally, I constructed bounding boxes to cover the area of each blob detected.  


### Test images with their heatmaps and bounding boxes:

![alt text][image5]


---

### Discussion

#### 1. Issues faced during the implementation of the project.  

The approach I took used the lesson techniques listed in the above goals / steps of the project.

The selected features (HOG, spatial) and Linear SVM worked fairly well, although I'm sure that further improvements could be made to optimize the classifier' accuracy. (e.g., tune parameters with a grid search)

Also, the pipeline might fail with faster moving vehicles or different lighting/weather conditions.

Lastly, to improve the project further we could implement "smarter" tracking of the detected vehicles (e.g., kalman filters), or optimize the sliding window search to improve the processing speed performance.
