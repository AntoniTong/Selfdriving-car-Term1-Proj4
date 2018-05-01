## Advanced Lane Lines Project 

### This project took a step forward from the earlier lane finding project, by implementing image perspective calibration and color filtering to better detect lane line. A search algorithm was also implemented to estimate the relative vehicle position, and the lane curvature.

---

**Advanced Lane Finding Project**

The goals / steps of this project are the following:

* Compute the camera calibration matrix and distortion coefficients given a set of chessboard images.
* Apply a distortion correction to raw images.
* Use color transforms, gradients, etc., to create a thresholded binary image.
* Apply a perspective transform to rectify binary image ("birds-eye view").
* Detect lane pixels and fit to find the lane boundary.
* Determine the curvature of the lane and vehicle position with respect to the center.
* Warp the detected lane boundaries back onto the original image.
* Output visual display of the lane boundaries and numerical estimation of lane curvature and vehicle position.

[//]: # (Image References)

[image1]: ./output_images/undistort_output.png "Chessboard Corners Detection"
[image2]: ./output_images/undistort_chessboard.png "Undistorted Chessboard"
[image3]: ./output_images/undistort_road.png "Undistorted road view"
[image4]: ./output_images/warped_image.png "Warped road"
[image5]: ./output_images/color_spaces.png "Color gradient and color spaces filterings"
[image6]: ./output_images/lane_find_new.png "Find new lanes"
[image7]: ./output_images/lane_find_nearest.png "Converging detected lanes"
[image8]: ./output_images/lane_find_nearest_result.png "Converging detected lanes"


[image10]: ./test_images/test1.jpg "Road Transformed"
[image13]: ./output_images/binary_combo_example.jpg "Binary Example"
[image14]: ./output_images/warped_straight_lines.jpg "Warp Example"
[image15]: ./output_images/color_fit_lines.jpg "Fit Visual"
[image16]: ./output_images/example_output.jpg "Output"
[video1]: ./project_video.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---


### Camera Calibration
### Python script `lane_finding.ipynb

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the second code block of the script

I start by loading the chessboard photos. The black and white contrast helps algorithm to identify corners of the chess board, for which I used `cv2.findChessboardCorners()`. The following figure shows the detected corners illustrated using `cv2.drawChessboardCorners()`.
![alt text][image1]
The significance of finding reference points, like those corners of a chessboard, is to correct image distortion caused by camera lenses.  As shown in the following figure.
![alt text][image2] 

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

Using the same method, I undistorted an image from vehicle camera.
![alt text][image3]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.


   
To detect lane lines, a helpful trick is to apply image filters so that pixels associated with lane lines will be stand out. The color gradient filter `cv2.Sobel` calculates image color derivatives in a specified direction. In our case, because lane lines are generally drawn vertically, calculate color gradient in x direct really helps pickup pixels associated with lane lines. 
In addition, filters associated with different color spaces ('HLS','LUV','Lab') help highlight lane lines as well. By a combination of multiple filters and playing with the thresholds, I was able to consistently detect lane line pixels:

   ![alt text][image5]
   
The code is located in the 5th block of the script.


#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

Because using a vehicle front camera, we are looking at the read from an angle. It is very beneficial to transfer the perspective to a bird view, in which the read is viewed from the top. To do that, I used a image warping function `M = cv2.getPerspectiveTransform(src, dst)` and 
    `warped = cv2.warpPerspective(undist, M, img_size)`. They transfer image perspective based on proposed reference point from both source image `src` and destination image `dst`. The following images show views of a road before and after warping.
    ![alt text][image4]
    
The code is located in the 4th block of the script. The source and destination codes are listed as the following

```python
    src_b_l = [220,720]
    src_b_r = [1110, 720]
    src_t_l = [571, 468]
    src_t_r = [720, 468]

    dst_b_l = [200,720]
    dst_b_r = [1000, 720]
    dst_t_l = [200, 0]
    dst_t_r = [1000, 0]
    
    
    src = np.float32([src_t_l,src_t_r,
                      src_b_r,src_b_l])
    dst = np.float32([dst_t_l,dst_t_r,
                      dst_b_r,dst_b_l])
```



#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

With the detected lane lines, I then used a search algorithm to look through 9 vertically stacked windows, to identify pixels that associated with the left and right lane lines.

![alt text][image6]


If the algorithm already processes detected lane line curvatures, the search becomes ealier by just looking at a narrower region of pixels, outlined by search margins:

```python
win_xleft_low = leftx_current - margin
win_xleft_high = leftx_current + margin 
win_xright_low = rightx_current - margin
win_xright_high = rightx_current + margin
```
![alt text][image7]
With detected pixels, I then use curve fitting function `np.ployfit()` to fit a second order polynomial function.

The code is located in the 6th block of the script.



#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to the center.

In the same code block, I calculated the lane line radius by using the formula:

```Python
left_curverad = 
	((1+ 
	(2*left_fit_cr[0]*np.max(ploty)
	+ left_fit_cr[1])**2)**1.5)
	/np.absolute(2*left_fit_cr[0])
right_curverad = 
	((1 + 
	(2*right_fit_cr[0]*np.max(ploty) 
	+ right_fit_cr[1])**2)**1.5) 
    /np.absolute(2*right_fit_cr[0])
```
    



#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

In the same code block, I plotted lane lines and their enclosed spaces. I also displayed the estimated turn radius and the relative location of the vehicle to the center of the road:

![alt text][image8]
---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./result.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

I tried on the challenge video, and it failed badly. To improve, the filtering and searching algorithm should be more intelligent. Because roads may curve to different directions, especially in steep turns, the color gradient algorithm should be more intelligent. Also, when a camera view switches between dark and bright, or even day and night, the color space filtering should adaptively normalize the image. 

In addition, even though the video algorithm stored the detected curvatures for the past n iterations, it still performs poorly when the car is driving through some poorly marked areas. Perhaps a more adaptive method is to implement a time series RNN, to predict lane lines.