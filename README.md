# Udacity Self-Driving Car Engineer Nanodegree
# Project: Advanced Laneline Detection
---

### Summary of Project
In this project, a lane line detection algorithm is developed in Python to detect curvature lane lines and the within lane region in videos shot by front facing camera in car. To achieve the goal, the following steps are taken:

* Compute the camera calibration matrix and distortion coefficients given a set of chessboard images.
* Apply a distortion correction to raw images.
* Use color transforms and gradients detection to create a thresholded binary image.
* Apply a perspective transform to rectify binary image or birds-eye view.
* Detect lane pixels and fit to find the lane boundary.
* Determine the curvature of the lane and vehicle position with respect to center.
* Warp the detected lane boundaries back onto the original image.
* Output visual display of the lane boundaries and numerical estimation of lane curvature and vehicle position.

The algorithm is implemented and tested on different highway driving videos, and successfully demonstrates the detection of curvature lane lines and lane regions under different lane line color and lighting conditions.

The code related to this project is provided in the file of AdvLaneLine.ipynb. The following section describes the method applied in each steps, with demonstrative examples. The code corresponding to each section is labeled in the ipynb file. 

---

### Camera Calibration (Code in Step 1: Camera calibration)

The camera calibration is started by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![alt text](https://github.com/davidsky900/SelfDrivingCar-AdvancedLanelineDetection/blob/master/output_images/undistort10.png)

---

### Pipeline for single image

#### Distortion correction (Code in Step 2: Undistort test images)
The image to be detected with laneline is first processed with distortion correction. The calibration matrix and calibration distance are obatained from the camera calibration process described above. To demonstrate this step, I will describe how I apply the distortion correction to one of the test images like this one:
![alt text](https://github.com/davidsky900/SelfDrivingCar-AdvancedLanelineDetection/blob/master/output_images/undistortTest3.png)
By comparing the signs on the left side of the two images, one can tell the distortion is corrected. 

#### Binary transformation (Code in Step 3: Binary transformation)
The second step is to transform the image after distortion correction into binary mode to extract the laneline features as much as possible. There are two types of filters used. One type of filters use different color channels of the images to captures white and yellow lines with rejecting the luminate changing as much as possible. Yellow filter, L channel in HLS color space, S channel in HLS space are used. The other type of filter is the Sobel filter that captures the gradient change of the pixel intensity. I used a combination of color and gradient fitlers plus thresholds.  Here's an example of my output for this step.  
![alt text](https://github.com/davidsky900/SelfDrivingCar-AdvancedLanelineDetection/blob/master/output_images/binaryTransform6.png)
The left subplot is the undistorted image, the middle subplot is the features extracted by Sobel filter and channel S filter for demonstrative purpose, the right subplot is the binary image with all fitlers output combined.

#### Perspective transformation (Code in Step 4: Perspective transform)
The perspective transformation is processed by transforming four specified source points into four specified destination points. The quality of transformation is determined by how parallel the lanelines are in the transformed images. 
This selected source and destination points are listed below （from top left to bottom left in clockwise direction:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 435, 560      | 275, 560        | 
| 895, 560      | 1055, 560      |
| 1180, 720     | 1055, 720      |
| 200, 720      | 275, 720        |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.
![alt text](https://github.com/davidsky900/SelfDrivingCar-AdvancedLanelineDetection/blob/master/output_images/perspectTransform0.png)
![alt text](https://github.com/davidsky900/SelfDrivingCar-AdvancedLanelineDetection/blob/master/output_images/perspectTransform5.png)

#### Identify and fit lane lines (Code in Step 5.1: Identify and fit lane lines )
The lane lines are find by searching the image from bottom to top in a sliding window fashion. In each sequence of search, the histogram of the image region is computed and the two regions (one for left and right side) where the most points are clustered are identified. Those two identified regions are served as the starting positions for the next round of search. After the sliding window search is performed, all the points within the hot regions (green boxes) are used to fit 2nd order polynomial for the lane lines. The results are shown below: 
![alt text](https://github.com/davidsky900/SelfDrivingCar-AdvancedLanelineDetection/blob/master/output_images/lanelineDetection4.png)
The left subplot plot is the image before persepective transformation, the middle subplot is the image with hot regions identified and the fitted lane lines (yellow lines)

#### Calculate radius of curvature and vehicle postion (Code in Step 5.2: Calculate radius of curvature and vehicle position)
The ratius of the curvature is caluclated by first convert the pixs of the fit lane lines in meters, and then calculated with equation with 2nd order polynomial. The deviation of vehicle position is compared the with the center of the road and the center of the image. 

#### Draw lane region (Code in Step 5.3: Inverse perspective transformation)
The region of the lane is highled with green color on the undistorted image by transform the topview lane region using the reverse transform matrix.  Here is an example of my result on a test image:
![alt text](https://github.com/davidsky900/SelfDrivingCar-AdvancedLanelineDetection/blob/master/output_images/lanelineDetection5.png)
The left subplot plot is the image before persepective transformation, the right subplot is the image with identified lane regions (green area). The radius of curvature and the vehicle position are printed as well.

---

### Pipeline (video)

#### The final video of the project is provided below. 
The overall performance is reasonable with few frames to be jumped due to the change of lighting condition. Here's a [link to my video result](https://github.com/davidsky900/SelfDrivingCar-AdvancedLanelineDetection/blob/master/output_videos/video1_out.mp4)

---

### Discussion

#### Briefly discuss any challenges in the project and potential solutions. 
The approach in general is successful in detecting lane lines with curvature, which is an improvement compared to the lane line detection in a previous project. However, there are still some challenges and could potentially be addressed, as listed below

a. In the video, when the lighting condition changes abruptly, especially when encountering shadows of trees, the detection performance will decline. This might be improved by dynamically change the threshold of L channel at frame by frame basis. 

b. The car appears on the lane will interupt the lane line detection, . This can be addressed by removing the image paches that is identified to be vehicle, so that the disturbance caused by vehicles can be minimized.
