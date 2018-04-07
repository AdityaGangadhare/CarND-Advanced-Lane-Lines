
---

## Advanced Lane Finding Project
---
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

[image1]: ./output_images/calib.JPG "Undistorted"
[image2]: ./output_images/warped.JPG "Road Transformed"
[image3]: ./output_images/thresholded.JPG "Binary Example"
[image4]: ./output_images/processed.JPG "Warp Example"
[image5]: ./output_images/linefit.JPG "Fit Visual"
[image6]: ./output_images/all.JPG "Output"
[video1]: ./project_video.JPG "Video"


---

### Camera Calibration

For finding lane lines which are curved we need images which are distortion free. Distorted images can be easily corrected usin test images. Samples of chess board images captured with the same camera which is used to record video are used for camera calibration.

The code for this step is contained in the second code cell of the jupyter notebook located in "./AdvancedLaneFinding.ipynb".  

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![alt text][image1]

### Pipeline (single images)

#### 1. Removing Distortion and Perspective Transform.

There are two main steps to this process: use chessboard images to obtain image points and object points, and then use the OpenCV functions `cv2.calibrateCamera()` and `cv2.undistort()` to compute the calibration and undistortion. The locations of the chessboard corners were used as input to the OpenCV function calibrateCamera to compute the camera calibration matrix and distortion coefficients. This camera matrix and distortion coefficients were used to remove distortion in the images.
Next the undistorted image is transfored to 'Bird's Eye View' of the road in which only lane lines are focused and appear relatively parallel to each other using `cv2.warpPerspective()`.

Source and destination points used:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 200,720      | 300, 720        | 
| 600, 447      | 300, 0      |
| 679,447     | 900, 0      |
| 1100,720      | 900, 720        |

![alt text][image2]

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

#### 2. Thresholding.

In this step I have converted the image to HLS colorspace to highlight only the lane lines. I used thresholding in S and L channels of the image but lane lines were getting distarcted due to shadows.
To overcome this I used a combination of color and gradient thresholds(sobel in x direction) to generate a binary image.

![alt text][image3]
![alt text][image4]


#### 3. Fitting a polynomial to the lane lines

After processing the image I was able to generate a  combined binary image having pixels belonging to lane lines. The next step was to fit a polynomial to each lane line, which was done by:

- Identifying peaks in a histogram of the image to determine location of lane lines.
- Identifying all non zero pixels around histogram peaks using the numpy function numpy.nonzero().
- Fitting a polynomial to each lane using the numpy function numpy.polyfit().

After fitting the polynomials I was able to calculate the position of the vehicle with respect to center.

![alt text][image5]

#### Final Output

![alt text][image6]

Here's a [link to my video result](./test_output_videos/project_video.mp4)

---

### Possible Limitations

The video pipeline developed in this project did a fairly robust job of detecting the lane lines in the test video provided for the project, which shows a road in basically ideal conditions, with fairly distinct lane lines, and on a clear day. 

It is relatively easy to finetune a software pipeline to work well for consistent road and weather conditions, but what is challenging is finding a single combination which produces the same quality result in any condition is difficult. 
