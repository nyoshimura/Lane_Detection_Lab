## Advanced Lane Finding Project

Contents are the following:

1. Camera Calibration
2. Pipeline (test images)
3. Pipeline (video)
4. Discussion

[//]: # (Image References)

[image1]: ./output_images/calib.png "Undistorted"
[image2]: ./output_images/calib_driving_img.png "Road Transformed"
[image3]: ./output_images/gradient_binary.png "Binary Example"
[image4]: ./output_images/hls_binary.png "hls Example"
[image5]: ./output_images/lane_features.png "Fit Visual"
[image6]: ./output_images/warp.png "Warped example"
[image7]: ./output_images/warp_lane_features.png "Warped feature example"
[video1]: ./project_video_output.mp4 "Video"

---

### 1. Camera Calibration

##### (1/1). Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the first & second code cell of the IPython notebook located in "./P4_Nobuto.ipynb".

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result:

![alt text][image1]

### 2. Pipeline (single images)

##### (1/6). Provide an example of a distortion-corrected image.
![alt text][image2]

##### (2/6). Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.
I used a combination of color and gradient thresholds to generate a binary image.  Here's an example of my output for this step. At first, I tried sobel operator for x & y, magnitude of gradient, direction of gradient, and fused one.

![alt text][image3]

Then, I realized that Applying sobel (x) is almost same with combined one, so I decided to use sobel operator. After that, I combined with binary image from color transform. I used HLS color space because S is suitable to detect yellow line as follows:

![alt text][image4]

##### (3/6). Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes a function called `warper()`.  The `warper()` function takes as inputs an image (`img`), as well as source (`src`) and destination (`dst`) points.  I chose the hardcode the source and destination points in the following manner:

```
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

![alt text][image6]

##### (4/6). Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

Then I did some other stuff and fit my lane lines with a 2nd order polynomial like this:

![alt text][image7]

Identifing lane-line pixels and fit their positions are included in same function:
```
blind_search(bird_binary, nwindows=9, margin=60, minpix=50)
```

##### (5/6). Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

I assumed that the lane is about 30 meters long and 3.7 meters wide, and convert radius of curvature and the position from image space to world space.

##### (6/6). Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

![alt text][image5]

---

### 3. Pipeline (video)

##### (1/1). Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./project_video_output.mp4)

---

### 4. Discussion

##### (1/1). Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

In this algorithm, base point is derived by np.argmax(), and position is calculated by mean. But there are lots of pattern of lane in the world, and if right / left lane is not single line but multiple lines, this algorithm may not be suitable.
