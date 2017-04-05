## Advanced Lane Finding Project ##

The goals / steps of this project are the following:

- Compute the camera calibration matrix and distortion coefficients given a set of chessboard images.
- Apply a distortion correction to raw images.
Use color transforms, gradients, etc., to create a thresholded binary image.
- Apply a perspective transform to rectify binary image ("birds-eye view").
- Detect lane pixels and fit to find the lane boundary.
- Determine the curvature of the lane and vehicle position with respect to center.
- Warp the detected lane boundaries back onto the original image.
- Output visual display of the lane boundaries and numerical estimation of lane curvature and vehicle position.

### Camera Calibration ###

#### Camera calibration and distortion correction ####

The code for this task is implemented in the second and third cell of the IPython notebook located in "./p4.ipynb". The first step is to prepare the "object points" which are actually the (x,y,z) coordinates of the chessboard corners in the world. New points are appended to `objpoints` every time i successfully detect  the chessboard corners in a test image (calibration image). The `imgpoints` are the (x,y) pixel position of the chessboard corners and new data is appended with every successful chessboard detection. 

Next i use `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `calibrate_camera` function (cell #3). After successful calibration the new images look like this:

![calibration](http://i.imgur.com/5691uQH.jpg)

### Pipeline (single images)

#### 1. Example of a distortion-corrected image.
After the calibration and distortion coefficients have been found i apply the same function to the test images from the car camera and validate the result. The code responsible for this is located in the cell id: 4.  An example of this step can be shown below:

![Undistort test image](http://i.imgur.com/xOnsoVc.jpg)

#### 2. Color transforms, gradients and methods to create a thresholded binary image. 

The next step is to create a binary image out of the color one that was taken from the car camera. 

The pipeline for creating the final binary is:

- Apply sobel operator for edge detection calculating gradient in x direction with specific threshold (cell id: 5)
- Apply sobel operator for edge detection calculating gradient in y direction with specific threshold (cell id: 5)
- Apply sobel operator for edge detection calculating the total gradient with specific threshold (cell id: 5)
- Apply sobel operator for edge detection calculating the direction of the gradient with specific threshold (cell id: 5)
- Apply HLS color transformation using the S channel and thresholding it. (cell id: 7)
- The final stage combines all previous results to one binary image aggregating all the usefull information (cell id: 8)

The final binary can be seen below:

![Final binary](http://i.imgur.com/pLztpUs.jpg)

#### 3. Perspective transform application.

The code for my perspective transform includes a function `corners_unwarp()`, which appears in the 9th code cell of the IPython notebook. The `corners_unwarp()` function takes as inputs an image (img), as well as source (src) and destination (dst) points along with the calibration and distortion coefficients (mtx, dist). I chose to hardcode the source and destination points resulted in:

<table>
	<tr>
        <td>Source</td>
		<td>Destination</td>
    </tr>
    <tr>
        <td>(593,450)</td>
		<td>(320,0)</td>
    </tr>
<tr>
        <td>(688,450)</td>
		<td>(960,0)</td>
    </tr>
<tr>
        <td>(1130,720)</td>
		<td>(960,720)</td>
    </tr>
<tr>
        <td>(185,720)</td>
		<td>(320,720)</td>
    </tr>
</table>

The purpose of the `corners_unwarp()` function is to transform the image to bird's eye view. Using the source and destination polygon coordinates i obtained the matrix `M` that maps them into each other using the `cv2.getPerspective` function. The inverse matrix `Minv` will be used for the inverse operation later in the algorithm.

After plotting these spots on a test image with parallel lanes i corrected the coordinates so the lines drawn on the warped image was parallel.

![Calibration of src and dst points](http://i.imgur.com/pI1S6LT.jpg)

#### 4. Identification of lane-line pixels and fitting their positions with a polynomial

After binarizing the image and transforming it to bird's eye view the lane lines can be seen clearly. The algorithm takes as input this binarized and warped image and proceeds according to the steps below:

- Compute the histogram of the bottom half of the image (more clear)
- Identify the two peaks (Indication where the lane lines are located)
- Apply sliding windows using as a starting point the location of the two peaks. The number and dimensions of windows are parameterized. For every pass the pixels with value 1 within the window region are consodered to form the lane. At the end all the identified pixels are saved and the lane is indentified.
- Fit a second order polynomial to each lane line using `np.polyfit`

The function utilizing the procedure above is called `find_lane_lines` and it is located in the cell id:11.

The result obtained is similar to this:

![Find lane lines](http://i.imgur.com/kWMDsVr.jpg)

#### 5. Calculation of radius of curvature of the lane and the position of the vehicle with respect to center.

The curvature calculation is implemented in the 13th cell in the function `find_curvature()`.  The function takes as arguments the fitting coefficient of the lanes from the warped image and after transforming them to the world space the calculation is done using the code: 

	left_curverad = ((1 + (2*left_fit_cr[0]*img_height + left_fit_cr[1])**2)**1.5) / np.absolute(2*left_fit_cr[0])

    right_curverad = ((1 + (2*right_fit_cr[0]*img_height + right_fit_cr[1])**2)**1.5) / np.absolute(2*right_fit_cr[0])
 
The vehicle position with respect to the center of the lanes is calculated in the function `find_offset_from_center()` located in the 13th cell.

#### 6. Example image of the result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in cell#14 in the function `draw_lane_lines()`.  Here is an example of my result on a test image:

![Draw lane lines example](http://i.imgur.com/H5PFOok.jpg)

---

### Pipeline (video)

#### 1. Provided link to the final video output.

Here's a link to my  [video result](./project_video_result.mp4)

