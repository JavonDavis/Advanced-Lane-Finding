## Advanced Lane Finding
[![Udacity - Self-Driving Car NanoDegree](https://s3.amazonaws.com/udacity-sdc/github/shield-carnd.svg)](http://www.udacity.com/drive)

In this project, I wrote a software pipeline to identify the lane line boundaries in a video. The source for the project can be found in the notebook file main.ipynb. 

**Advanced Lane Finding Project**

The main goals / steps of this project are the following:

* Compute the camera calibration matrix and distortion coefficients given a set of chessboard images.
* Apply a distortion correction to raw images.
* Use color transforms, gradients, etc., to create a thresholded binary image.
* Apply a perspective transform to rectify binary image ("birds-eye view").
* Detect lane pixels and fit to find the lane boundary.
* Determine the curvature of the lane and vehicle position with respect to center.
* Warp the detected lane boundaries back onto the original image.
* Output visual display of the lane boundaries and numerical estimation of lane curvature and vehicle position.

[//]: # (Image References)

[image_undistorted_calibration]: ./readme_images/undistorted_calibration.png "Undistorted Calibration"
[image_undistorted_example]: ./readme_images/undistorted_example.png "Undistorted"
[image_example]: ./readme_images/image_example.png "Example Image"
[image_example_lanes_identified]: ./readme_images/image_example_lanes_identified.png "Example Image identified"
[image_points]: ./readme_images/image_points.png "Image Points"
[image_warped]: ./readme_images/image_warped.png "Image Warped"
[image_gradient]: ./readme_images/image_gradient.png "Image Gradient"
[image_hls]: ./readme_images/image_hls.png "Image HLS"
[image_rgb]: ./readme_images/image_rgb.png "Image RGB"
[image_color_gradient_combined]: ./readme_images/image_color_gradient_combined.png "Image Color+gradient"
[image_thresh_warped]: ./readme_images/image_thresh_warped.png "Image Threshold Warped"
[image_sliding_window]: ./readme_images/image_sliding_window.png "Image Sliding Window"



### Camera Calibration and Distortion Correction

The code for this step is contained in the first few code cells of the IPython notebook. The chessboard camera calibration technique is implemented in both the `load_points(images)` and `undistort_image(img, objpoints, imgpoints)` function to try to effectively calibrate the camera. First I started by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image. Thus, objp is just a replicated array of coordinates, and objpoints will be appended with a copy of it every time I successfully detect all chessboard corners in a test image. imgpoints will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.

I then used the output objpoints and imgpoints to compute the camera calibration and distortion coefficients using the cv2.calibrateCamera() function. I applied this distortion correction to a test image using the cv2.undistort() function and obtained this result:

![alt text][image_undistorted_calibrationn]


### Pipeline

The software pipeline accepts an image as input and returns that same image with the line lines identified and the area inbetween highlighted in green, like so:

![alt text][image_example] ![alt text][image_example_lanes_identified]

The pipeline is then fed images frame by frame from a video stream and builds a new video where the lane lines are identified throughout.

Below I'll describe the different steps the pipeline takes to achieve this in the order they were performed.

#### 1. Resize image

First to ensure that all the images are the same size as the images used to build the pipeline. This is a precautionary step to ensure the best results are produced by the pipline. 

### 2. Undistort the Image

As mentioned above the output objpoints and imgpoints are used to compute the camera calibration and distortion coefficients using the *cv2.calibrateCamera()* function. I applied this distortion correction to the image passed to the pipline using the *v2.undistort()* function, here's an example: 

![alt text][image_undistorted_example]

### 3. Perform a perspective transform

The code for my perspective transform includes a function called `warp(img)`, which appears in the 9th code cell of the IPython notebook.  The `warp(img)` function takes as inputs an image (`img`), and uses globally defined source (`src`) and destination (`dst`) points.  I chose the hardcode the source and destination points in the following manner:

```python
  src = np.float32(
      [[800, 500],
       [1045, 700],
       [370, 700],
       [550, 500]])

  dst = np.float32(
      [[900, 200],
       [900, 650],
       [200, 650],
       [200, 200]])
```


This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 800, 500      | 900, 200       | 
| 1045, 700     | 900, 650      |
| 370, 700    | 200, 650      |
| 550, 500      | 200, 200        |


The source points were chosen by eyeballing a few of the sample images and playing with a few options to get the best result.
Here you can see the 4 points chosen plotted on a sample image, 

![alt text][image_points]

The destination points were also chosen by eyeballing a set of rectangular points in the same space the image is originally ploted. I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![alt text][image_warped]

### 4. Gradient and color thresholds

These thresholds were applied in an effort to clearly identify the important features of lane lines in images. I used a combination of color and gradient thresholds to generate a binary image thresholding steps demonstrated from the 11th to the 35th code cells in the iPython Notebook. 

#### Gradient Threshold

When applying the gradient threshold various aspects of the gradient were considered such as the gradient with respect to both the x and y direction, the magnitude and the direction of the gradient. These were combined in a way to yield the best result. 
Here's an example for how the image looked after Gradient Thresholding was applied,

![alt text][image_gradient]

#### Color Threshold

Two color spaces were explored, RGB and HLS. A sample of both can be seen below,

RGB

![alt text][image_rgb]

HLS

![alt text][image_hls]

Both channels were tested with a number of images and the S channel seems to have performed the best.

This channel was then combined with the previous gradient threshold to produce a final binary image that would ideally have the both lane lines clearly identified. See example below,

![alt text][image_color_gradient_combined]

Note this would be performed on our warped image so in the pipleine it would more likely look like this,

![alt text][image_thresh_warped]

### 5. Fitting Lane Lines

Then I used the sliding window histogram technique and fit my lane lines with a 2nd order polynomial kinda like this:

![alt text][image_sliding_windows]
