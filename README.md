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
[image_histogram]: ./readme_images/image_histogram.png "Image Histogram"
[image_sliding_window]: ./readme_images/image_sliding_window.png "Image Sliding Window"
[image_final]: ./readme_images/image_final.png "Image Final"
[project_video]: ./output_videos/project_video.mp4 "Project Video"
[challenge_video]: ./output_videos/challenge_video.mp4 "Challenge Video"
[bad_output_highlighted]: ./readme_images/bad_output_highlighted.png "Bad Output"




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

### 5. Finding and fitting Lane Lines

#### Lane Finding Method: Peaks in a Histogram

I first take a histogram along all the columns in the lower half of the image like this,

```pyton
histogram = np.sum(binary_warped[binary_warped.shape[0]//2:,:], axis=0)
plt.plot(histogram)
```

Here's a sample of what the result might look like,

![alt text][image_histogram]

With this histogram I am adding up the pixel values along each column in the image. In my thresholded binary image, pixels are either 0 or 1, so the two most prominent peaks in this histogram will be good indicators of the x-position of the base of the lane lines. I can use that as a starting point for where to search for the lines. From that point, I can use a sliding window, placed around the line centers, to find and follow the lines up to the top of the frame.

Here is [a link](https://youtu.be/siAMDK8C_x8) to a short animation showing this method.

After all the points for the left line and the right line are identified they are then fit to a second order polynomial using the `np.polyfit` function. Here's an example of the what the result of this step would look like,

![alt text][image_sliding_window]

The exact code implementation is defined in the `sliding_window` function which can be found in the 36th code cell in the iPython Notebook. 

### 6.  Finding the Radius of curvature

The radius of curvature is the radius of the circular arc which best approximates the curve at a point. Calculating this would help in understanding if my polynomial makes sense. This calculation along with the distance from the center was done in the 40th code cell in the iPython Notebook. This radius was caluated for both the left and right lane lines and was printed out back on the image

### 7. Final plot

All of this with the lines identified was then plotted back on the original image and then used draw a shape which should line up with the lane lines. This was shown earlier at the beginning but here is another example of what the final result would look like, 

![alt text][image_final]

### 8. Video with pipeline applied

Here's the pipeline when applied to the project_video.mp4 file,

![alt text][project_video]

### 9. Problems Faced and Improvements

A big problem I had was identifying the lane lanes for example when there was actually none on the road after warping the image as this is the case in at some points in the video. I would end up with lines that fit like this, 

![alt text][bad_output_highlighted]

and no matter how hard I tried the line on the right just could not be properly identified with the gradient and color thresholds. To combat this I used an implementation I found online that tries to 'remember' the previous lines and use that as a kind of best fit for a certain number of frames.

The pipeline doesn't seem to perform well at sharp turns and also when there are a lot more distractions on the road it definately underperforms, you can see examples of this here on this video, 

![alt text][challenge_video]

So there's definately room for improvement there. 
