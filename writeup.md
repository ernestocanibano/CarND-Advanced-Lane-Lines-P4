## Advanced Lane Finding

### Template by Ernesto Cañibano.

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

[image1]: ./examples/image1.png "Corners detection"
[image2]: ./examples/image2.png "Camera calibration result"
[image3]: ./examples/image3.png "Undistorted image"
[image4]: ./examples/image4.png "Lines detection"
[image5]: ./examples/image5.png "Perspective transformation"
[image6]: ./examples/image6.png "Binary image"
[image7]: ./examples/image7.png "Line detection example"

[image8]: ./output_images/straight_lines1.png "Straight lines 1"
[image9]: ./output_images/straight_lines2.png "Straight lines 2"
[image10]: ./output_images/test1.png "test1"
[image11]: ./output_images/test2.png "test2"
[image12]: ./output_images/test3.png "test3"
[image13]: ./output_images/test4.png "test4"
[image14]: ./output_images/test5.png "test5"
[image15]: ./output_images/test6.png "test6"
[image16]: ./output_images/test8.png "test8"
[image17]: ./output_images/test9.png "test9"


## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README

All the code and the examples are included in the Jupyter Notebool [P4.ipynb](./P4.ipynb).

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the code cells 1 and 2 of the IPython notebook located in "./P4.ipynb".  

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

In the following images it is possible to see the corners detection in the chessboards.

![alt text][image1]

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![alt text][image2]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

To demonstrate this step, I will describe how I apply the distortion correction to one of the test images like this one:

![alt text][image3]

The code to generate the images above is located in the cell 3. I used the function `cv2.undistort()` with the matrix `mtx` and `dist` obtained during calibration proccess.

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

This process is poreformed by the function `thresholdImage()` which code is in cell 16. I used a combination of color 
and two gradient thresholds to generate a binary image. The proccess is summarized here:
* Grandient Threshold of the grayscale image. Works fine to detect the white lanes.
* Gradiente threshold of the S-channel of the image converted to HLS colormap. It's good detecting yellow lanes.
* Color threshold of the H-channel of the HLS image. I can't filter vertical lanes which are not marks of the road.

Here are two examples of my output for this step.  

![alt text][image4]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes a function called `warper()` and another one `unwarper()`, which 
appear in cell 18.  The `warper()` functions takes as inputs an image (`img`), and uses two variables (`src`) and destination 
(`dst`) points.  I chose the hardcode the source and destination points in the following manner:

```python
src = np.float32( [[583,460], [207,binary_images[0].shape[0]], [1100,binary_images[0].shape[0]],[698,460]])
dst = np.float32( [[327,0], [327,binary_images[0].shape[0]], [980,binary_images[0].shape[0]],[980,0]])
```

This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 583, 360      | 327, 0        | 
| 207, 720      | 327, 720      |
| 1100, 720     | 980, 720      |
| 698, 460      | 980, 0        |

To calculate the origin and destination points I did several measures over the first test image.
I verified that my perspective transform was working as expected by drawing the `src` and `dst` 
points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.
The code to generate the following images is in cell 19.

![alt text][image5]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

This proccess is performed by the function `drawLineBoundaries()` which code is in cell 20. This function received the warped image. Two examples of the input images are showed in the image bellow.

![alt text][image6]

The function receive as parameter the class `Lines`. This class is used to save the line data of the previous frame
of the video. The funcion follows the same step as in the lecture 13 of the nanodegree but with the following modifications.

* If data of the previous frame is available, the calculation of the lines is done using the previous data instead of the slicing windows method.
* If in the current frame any point is detected, we use the data of the previous frame.
* At the end of the function a sanity check is done. If the parameters of curvature, position and width of the lane present any abrupt change with the previous frame, the current frame is discared.

In the following image is possible to see al the steps of the proccess in detail.

![alt text][image7]

The data output (lines, points, image...) are stored in an object of the class `Lines()` to be used later in the pipeline.

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

All the data, curvature, car position and width of the lane are drawn in yellow color over the output image. The calculation
of these parameters are peformed by the function `drawLineBoundaries()` (cell 20) and painted in the image by the function
`addTextToImage()` (cell 24). To transform the lenghts from pixel to meters y use the first test image and assumed a width of the lane
of 3.7 meters.

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in function `addPoligonToImage()` in cell 24. Here is an example of my result on a test images. I added
two additonal images to test the *challenge video*.

![alt text][image8] ![alt text][image9]
![alt text][image10] ![alt text][image11]
![alt text][image12] ![alt text][image13]
![alt text][image14] ![alt text][image15]
![alt text][image16] ![alt text][image17]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./project_video_output.mp4).

I will use the pipeline with the [challenge video](./challenge_video_output.mp4).

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

The pipeline fails when another vertical lines appear in the road: for example when the asphalt has two different colors, black wheel marks... 
I solved this with the sanity check. To improve the pipeline I think i will have to improve de detection of the lines (function `thresholdImage()` in cell 16). With this i think it would be possible to detect the lines in the third video.
