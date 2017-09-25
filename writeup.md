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

[linethreshold]: ./test_images/line_threshold.png "line threshold"
[colorthreshold]: ./test_images/color_threshold.png "color threshold"
[stackedcombined]: ./test_images/stacked_combined.png "stacked combined"

[thresholded1]: ./output_images/thresholded_image1.png "thresholded1"
[thresholded2]: ./output_images/thresholded_image2.png "thresholded2"
[thresholded3]: ./output_images/thresholded_image3.png "thresholded3"
[thresholded4]: ./output_images/thresholded_image4.png "thresholded4"
[thresholded5]: ./output_images/thresholded_image5.png "thresholded5"
[thresholded6]: ./output_images/thresholded_image6.png "thresholded6"

[pipeline1]: ./output_images/pipeline1.png "pipeline1"
[pipeline2]: ./output_images/pipeline2.png "pipeline2"
[pipeline3]: ./output_images/pipeline3.png "pipeline3"
[pipeline4]: ./output_images/pipeline4.png "pipeline4"
[pipeline5]: ./output_images/pipeline5.png "pipeline5"
[pipeline6]: ./output_images/pipeline6.png "pipeline6"

[search_area]: ./output_images/search_area.jpg "search area"
[find_window_centroid]: ./output_images/find_window_centroid.jpg "find window centroid"

[drawnlines]: ./output_images/drawn_lines.jpg "drawn lines"

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

#### 1. Thresholded Binary Image

I created a pipeline that does the following to create a binary image:

1. Undistort an image
2. Take a line threshold of the image
3. Take a color threshold of the image

The undistort operation is detailed above, with an example image.

To create a line threshold, we utilize the Sobel operator in addition to a magnitude and directional thresholds. We tuned these filters to achieve a result like the following:

| Original Image | Line Threshold |
|:--------:|:------------:|
| ![left][calibration4] | ![center][linethreshold] |

As this image makes clear, the line threshold alone has a difficult time detecting lane lines that don't have a high gradient. The yellow lane line in this photo is barely visible.

To improve upon this result, we add an additional color thresholding function as well. The color threshold function converts the image into two separate HLS and HSV color spaces, which are tuned and then combined. The result is an image that can highlight lane lines in situations that are much more difficult for the line threshold alone:

| Original Image | Color Threshold |
|:--------:|:------------:|
| ![left][calibration4] | ![center][colorthreshold] |

Below is an image that shows the two different thresholding techniques combined on an undistorted test image, and the resulting binary image:

![stacked_combined][stackedcombined]

Below are the results of creating the thresholded binary image on the test images provided by Udacity:

![1][thresholded1]
![2][thresholded2]
![3][thresholded3]
![4][thresholded4]
![5][thresholded5]
![6][thresholded6]

#### 2. Perspective Transform

I created a function to warp images called, fittingly, `warp_image`. This function takes an image as an input and uses the following `src` and `dst` calculations warp an input image to make the lane lines roughly parallel:

```python
    img_size = (image.shape[1], image.shape[0])
    
    bottom_width = .62
    middle_width = .07
    height_pct = .62
    bottom_trim =  .935

    src = np.float32([[img.shape[1]*(0.5-middle_width/2), img.shape[0]*height_pct],
                      [img.shape[1]*(0.5+middle_width/2), img.shape[0]*height_pct],
                      [img.shape[1]*(0.5-bottom_width/2), img.shape[0]*bottom_trim],
                      [img.shape[1]*(0.5+bottom_width/2), img.shape[0]*bottom_trim]])

    offset = img_size[0]*.2
    
    dst = np.float32([[offset, 0],
                      [img_size[0]-offset, 0],
                      [offset, img_size[1]],
                      [img_size[0]-offset, img_size[1]]])
```

This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 595, 446      | 256, 0        | 
| 684, 446      | 1024, 0      |
| 243, 673     | 256, 720      |
| 1037, 673      | 1024, 720        |

To verify that this code resulted in roughly parallel lane lines, I ran the function on the test images provided by Udacity, and plotted the results. The histogram chart also included shows that the lane lines are pretty well-detected in all of the test images:

![1][pipeline1]
![2][pipeline2]
![3][pipeline3]
![4][pipeline4]
![5][pipeline5]
![6][pipeline6]

#### 3. Finding Lane Pixels

Using the code base provided by Udacity, we implement a sliding window technique to find lane lines. This algorithm works by taking the maximum values of the histogram from the left and right sides of an image and using that as a starting point, and then searching in rectangles within certain margins through the rest of the image.

To visualize this algorithm, I've outputted an image called `find_window_centroid.jpg` that shows the left lane, right lane, and the searched rectangles.

![find window centroid][find_window_centroid]

To make our search more efficient, if we found a lane in a previous iteration, then we search around that previous iteration. We used the accompanying function `draw_search_area` below to generate an image called `search_area.jpg` that shows the area around which we look for a new lane.

![search area][search_area]

#### 4. The `Lane` Class

We create a couple of helper classes to make our pipeline easier to run. We create a `Lane` class, which contains two `Line` instances. The Line class, as suggested by Udacity, holds values for each lane line, including previously found values that assist us in finding subsequent line values. The `Lane` class contains functions to find the left and right lane lines, process images, and perform sanity checks on subsequently found lane line images.

The `get_curvature` function within the `Lane` class calculates the curvature of the lane by converting the pixels to meters and fitting a second-order polynomial to the curve:

```python
    def get_curvature(self, y, fitx):
        # Define y-value where we want radius of curvature
        # I'll choose the maximum y-value, corresponding to the bottom of the image
        y_eval = np.max(y)
        # Define conversions in x and y from pixels space to meters
        # assume the lane is about 30 meters long and 3.7 meters wide
        ym_per_pix = 35/720 # meters per pixel in y dimension
        xm_per_pix = 3.7/800 # meters per pixel in x dimension

        fit_cr = np.polyfit(y * ym_per_pix, fitx * xm_per_pix, 2)
        curverad = ((1 + (2*fit_cr[0]*y_eval*ym_per_pix + fit_cr[1])**2)**1.5) / np.absolute(2*fit_cr[0])
        return curverad
```

The `Lane` class also contains a `sanity_check` function that checks to make sure that subsequently found lane lines make sense compared to previous lines. A valid new lane line must not differ from a previously found lane line in the following ways:

- The radius of curvature must be within 70% of the previously found curvature
- The base position of the lane (the x position of the lane at the highest y value, i.e., the lane position closest to the car), must not differ by 100 pixels or more, and
- The x position of the lane at the position furthest from the car must also not differ by 100 pixels or more.

There is the possibility that we decide a lane line in a new frame is not valid, and subsequent lane lines differ more and more from the previous frame. To ameliorate that potential situation, we introduce a variable consecutive_invalid to keep track of the number of consecutive times we found an invalid line. If we find 10 consecutive invalid lines, we use this newfound lane.
        
The position of the car is calculated in the `draw_lane` function, which takes the x values of the left lane and the right lane closest to the car, and calculates a difference between that and the center of the image, converting from the pixel space to meters.

The found lane is re-warped back and drawn onto an undistorted image, and the lane curvature and the car's distance from center is added as well. Here is an example image:

![drawn][drawnlines]

---

### Pipeline (video)

Here's a [link to my video result](./output.mp4)

---

### Discussion

This pipeline fairly well on the `project_video.mp4` file. There are some instances in the video where the lane lines jump a little bit, and this appears to be caused by the presence of shadows where the lane color becomes brighter than the typically black. The video `output.mp4` contains the output video.

Using this pipeline on the challenge video yielded poor results, as can be seen on the `challenge_output.mp4` video. To obtain better results on that set of images, we would like to tweak the parameters and use test images that are more similar to that video.

Further improvements might include an image mask, which would serve to eliminate some of the peripheral lines. In some of the instances, the left wall of the freeway would be strongly detected as a line, which might cause the detection to misinterpret the results in some cases. We tuned the transform parameters to avoid that as much as possible, but a mask around the lane and the horizon would make that even more robust.

Further improvements could be make by tuning the sanity check and the tuning parameters to be a bit more reliable around some of the more troublesome areas of the video, such as when the freeway color became lighter and included shadows.
