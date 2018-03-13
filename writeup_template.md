
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

[image1]: ./output_images/undistorted_chessboard.png "Undistorted"
[image2]: ./output_images/undistorted_highway.png "Road Transformed"
[image3]: ./output_images/color_gradient_with_lab.png "Binary Example"
[image4]: ./output_images/warped_example.jpg "Warp Example"
[image5]: ./output_images/sliding_window.jpg "Fit Visual"
[image6]: ./output_images/img_with_text.png "Output"
[image7]: ./output_images/pipeline_img_with_lanes.png "Pipeline with lanes"
[video1]: ./project_video_output.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Camera Calibration

#### 1. To correct the image distortion, first we need to do a camera calibration.
The steps to perform a camera calibration is to use the camera images that has easily identifiable partners, for example, use a chessboard. In this project, we will scan through 20 pictures took from various angles and distance to calculate the camera matrix and distortion coefficients. We used opencv `calibrateCamera()` function.
The code for this step is contained in the code cell 1 and 2 of the IPython notebook located in "carnd-advanced-lane-lines.ipynb".  

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (9, 6) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (9,6) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result:

![alt text][image1]

### Pipeline (single images)

#### 1. Here is an example of a distortion-corrected image of the highway.

To demonstrate this step, I apply the distortion correction to one of the test images like this one:
![alt text][image2]

#### 2. Methods to create a thresholded binary image.  

I used a combination of color gradient to generate a binary image (thresholding steps at lines #11-36 function `color_gradient()` in cell 11 of the notebook `carnd-advanced-lane-lines.ipynb`).
There are a couple of threshold parameters we can tune to get the optimal results of identifying the lane marks. One is the s_channel threshold and the other is the sobelx threshold. Combing the color gradient with the LAB b channel filter (threshold) achieved the best results to identify the yellow lines as well as the white lines on the bright pavements
Here's an example of my output for this step.  

![alt text][image3]

#### 3. Perspective Transform

The code for my perspective transform includes a function called `perspective_transfer()`, which appears in lines 21 through 25 in cell 12 of the jupyter notebook `carnd-advanced-lane-lines.ipynb`   The `perspective_transfer()` function takes as inputs an image (`img`).  I chose the (`src`) source and destination (`dst`) points in the following manner:

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
| 1126,720      | 960, 720      |
| 695, 460      | 960, 0        |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![alt text][image4]

#### 4. Identified lane-line pixels and fit their positions with a polynomial
First from the histogram, we find the base x of the left and right lane's starting points. Then we have a vertical sliding window to slide through the image and scan for the lane pixels. Here I used 9 sliding windows, set the margin to be 100 pixels and minimum number pixels found to recenter the window. After slide through 9 slice, I extract left and right line pixel positions, then I fit my lane lines with a 2nd order polynomial like this: (Notebook cell 14 line 45-90)

![alt text][image5]

#### 5. Calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

In this part, we define the meter per pixel in y dimension and x dimension. For x dimension, our lane is 640 pixel wide, which corresponding to about 3.7 meters. We first extract the pixels positions for left and right lanes. And then we fit the new polynomial to x, y in world space.  and then calculate the curvature of the left and right lane. I did this in lines 9 through 38 in my code in cell 16 of `carnd-advanced-lane-lines.ipynb`

#### 6. An example image of the result plotted back down onto the road such that the lane area is identified clearly.
I plotted back the lane area into the original image. I implemented this step in lines 21 through 23 in my code in cell 18 of `carnd-advanced-lane-lines.ipynb` in the function `draw_lanes()` and function `add_text()` in cell 19, line 9 to add the text on to the image.  From the fit, we calculate the fit x for left and right lane. Then we draw the lane onto the warped black image. After that, we warp the blank back to original image space using inverse perspective matrix `Minv`. Then we combine the result with the original image by `cv2.addWeighted()` Here is an example of my result on a test image:

![alt text][image6]

---

### Pipeline
#### 1 Pipeline with test images.
I applied the pipeline to the test images to show the lane lines on the image after going through the pipeline. Here is an example of the resulting image that shows the warped image from the original.

![alt text][image7]

#### 2. Link to the final video output.  

Here's a [link to my video result](./project_video_output.mp4)

---

### Discussion

#### 1. Discussion: problems / issues / approach

- The camera calibration, image undistortion are quite straightforward to implement. I tried sobel threshold, magnitude threshold, direction threshold for the line detection on the image. I also tried HLS and color gradient for the line detection. The color gradient is good for detect especially the yellow line in the testing images with dark pavement. However, for the bright pavement, we have difficulties identify the yellow line. So I also tried out the b channel from the LAB color space as suggested by the instructor. The b channel of the LAB color space is great for identify yellow line. Current pipeline works well on the project_video, however, there are some difficulties on the challenge video especially under the bridge with some shade and there is old lane with dark surface and the newly repaired lane with light surface. There is an apparent line been detected by the algorithm. This line (mark) is between the old surface and the new surface, it could be a common challenge for the self driving car. If I have more time to continue working on this issue, I would try more HLS and HSV color space to see how we can ignore this kind of line (not the lane line).
- Other area of improvement is to tuning the low pass filter to achieve an optimal smooth out of jittering of lane detection from frame to frame.
- Current implementation is using a fixed source and destination points for the perspective transformation. In the real world, highway surface can be tilted or cars can go down hill or up hill, the perspective transformation needs to be dynamic and adapt
