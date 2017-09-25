## Writeup Template

### You can use this file as a template for your writeup if you want to submit it as a markdown file, but feel free to use some other method and submit a pdf if you prefer.

---

**Advanced Lane Finding Project**

The goals / steps of this project are the following:

* Compute the camera calibration matrix and distortion coefficients given a set of chessboard images.
* Apply a distortion correction to raw images.
* Use color transforms, gradients, etc., to create a thresholded binary image.
* Apply a perspective transform to rectify binary image ("birds-eye view").
* Detect lane pixels and fit to find the lane boundary.
* Determine the curvature of the lane and vehicle position with respect to center.
* Warp the detected lane boundaries back onto the original image.
* Output visual display of the lane boundaries and numerical estimation of lane curvature and vehicle position.

[//]: # (Image References)

[calibration1]: ./camera_cal/calibration2.jpg "calibration image"
[calibration2]: ./camera_cal/calibration2.jpg-corners.jpg "calibration corners"
[calibration3]: ./camera_cal/calibration2_undistorted.jpg "undistorted image"
[calibration4]: ./test_images/test1.jpg "calibration test image"
[calibration5]: ./test_images/test1_undistorted.jpg "undistorted test image"

[pipeline1]: ./output_images/pipeline1 "pipeline1"
[pipeline2]: ./output_images/pipeline2 "pipeline2"
[pipeline3]: ./output_images/pipeline3 "pipeline3"
[pipeline4]: ./output_images/pipeline4 "pipeline4"
[pipeline5]: ./output_images/pipeline5 "pipeline5"
[pipeline6]: ./output_images/pipeline6 "pipeline6"

[thresholded1]: ./output_images/thresholded_image1 "thresholded1"
[thresholded2]: ./output_images/thresholded_image2 "thresholded2"
[thresholded3]: ./output_images/thresholded_image3 "thresholded3"
[thresholded4]: ./output_images/thresholded_image4 "thresholded4"
[thresholded5]: ./output_images/thresholded_image5 "thresholded5"
[thresholded6]: ./output_images/thresholded_image6 "thresholded6"

[search_area]: ./output_images/search_area.jpg "search area"
[find_window_centroid]: ./output_images/find_window_centroid.jpg "find window centroid"

[video1]: ./output.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README

All code is in the jupiter notebook file `advanced-lane-lines.ipynb`, located at the root of the project directory.

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for camera calibration is in the first code cell of the notebook. The code calibrates the camera using calibration images provided by Udacity and chessboard corners functions provided by cv2. A loop iterates through the calibration images, and the cv2 function `findChessboardCorners` function attempts to find the chessboard corners. The `objpoints` array below saves the three-dimensional coordinates of the chessboard corners, and the `imgpoints` array saves the pixel position of the corners. The chessboards were set to have 9x6 internal corners.

Three of the calibration images failed, but we were able to successfully calibrate the camera despite that. The images with the found corners are in the camera_cal/ folder, suffixed with `-corners.jpg`

| Original | Corners Found | Undistorted |
|:--------:|:------------:|:------------:|
| ![left][calibration1] | ![center][calibration2] | ![right][calibration3] |

Here is an example of undistorting one of the test images:

| Original Image | Undistorted |
|:--------:|:------------:|
| ![left][calibration4] | ![center][calibration5] |

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

To demonstrate this step, I will describe how I apply the distortion correction to one of the test images like this one:
![alt text][image2]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used a combination of color and gradient thresholds to generate a binary image (thresholding steps at lines # through # in `another_file.py`).  Here's an example of my output for this step.  (note: this is not actually from one of the test images)

![alt text][image3]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes a function called `warper()`, which appears in lines 1 through 8 in the file `example.py` (output_images/examples/example.py) (or, for example, in the 3rd code cell of the IPython notebook).  The `warper()` function takes as inputs an image (`img`), as well as source (`src`) and destination (`dst`) points.  I chose the hardcode the source and destination points in the following manner:

```python
src = np.float32(
    [[(img_size[0] / 2) - 55, img_size[1] / 2 + 100],
    [((img_size[0] / 6) - 10), img_size[1]],
    [(img_size[0] * 5 / 6) + 60, img_size[1]],
    [(img_size[0] / 2 + 55), img_size[1] / 2 + 100]])
dst = np.float32(
    [[(img_size[0] / 4), 0],
    [(img_size[0] / 4), img_size[1]],
    [(img_size[0] * 3 / 4), img_size[1]],
    [(img_size[0] * 3 / 4), 0]])
```

This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 585, 460      | 320, 0        | 
| 203, 720      | 320, 720      |
| 1127, 720     | 960, 720      |
| 695, 460      | 960, 0        |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![alt text][image4]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

Then I did some other stuff and fit my lane lines with a 2nd order polynomial kinda like this:

![alt text][image5]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

I did this in lines # through # in my code in `my_other_file.py`

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in lines # through # in my code in `yet_another_file.py` in the function `map_lane()`.  Here is an example of my result on a test image:

![alt text][image6]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./project_video.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

Here I'll talk about the approach I took, what techniques I used, what worked and why, where the pipeline might fail and how I might improve it if I were going to pursue this project further.  
