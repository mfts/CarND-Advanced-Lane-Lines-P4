# **Advanced Lane Finding** 

## Setup

### Installation

Runs Jupyter Notebook in a Docker container with `udacity/carnd-term1-starter-kit` image from [Udacity][docker installation].

```
cd ~/src/CarND-Trafic-Sign-Classifier-Project
docker run -it --rm -p 8888:8888 -v `pwd`:/src udacity/carnd-term1-starter-kit
```
Go to `localhost:8888`


## Reflection

### A. Camera Calibration
##### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image

The code for this step is contained in the second and third cell of the iPython notebook _pipeline.ipynb_. 

I start by creating object points with 3-dimensional coordinates. However because I assume that the z-plane = 0 I am left with a 2-dimensional chessboard, which is the same for each calibration image.

I run a OpenCV function `findChessboardCorners` in order to detect corners between black and white squares of the chessboard. Everytime corners are found it will be appended to to an _objectpoints_ array. Similarly, with every found corner a two coordinates in pixel position will be appeneded to an _imagepoints_ array.
Here is a result of the chessboard images with corners:

![alt text][chessboard_image_corners]

With the help of these object- and imagepoints arrays I can run a calibration function `cv2.calibrateCamera()` on an image taken with the same camera. Finally I use the calibration matrix (optionally also radial and tangetial vectors) to correct my image for camera distortions `cv2.undistort()`. 

### B. Pipeline (single images)
##### 1. Provide an example of a distortion-correction image
And here is a result of an undistorted street image:

![alt text][undistorted_image]

##### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a threshold binary image. Provide an example of a binary image result
From the lecture I saw how to use Sobel gradients used in Canny Edge detections. However, I wanted to test a pure color space approach. I tested around with color channels I've previously not heard of such as HSV/HSL-Spaces, LUV-Space and Lab-Space. I landed on utilizing only LUV and Lab because they both identified the **white** and **yellow** lines, respectively better than the RGB-Color Space for example. The pipeline fuction for applying a binary threshold to an image is called **apply_binary()**.

![alt text][binary_image]

##### 3. Desribe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.
The pipline function for performing a perspective transform to an image is called **warp()**.
Here the hardest part was finding an appropriate set of points on the image that creates a rectangle which can be transformed. After trying to do it in code, I opted for a separate program (actually Sketch - a drawing application) to put dots on my image. I landed on a rectangle that encapsulates all test images lane lines:
```
src = np.float32(
    [[490, 480],[800, 480],[1180, 640],[170, 640]])
dst = np.float32(
    [[0,0],[1280,0],[1280,720],[0,720]])
```
These are the source and destination rectangles for the image. All images used were 1280x720 pixels.
I set the rectangle on the straight test images and then applied it to all test images. Here's a look at a straight and curved test image:

![alt text][warped_straight_image]
![alt text][warped_curved_image]

##### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?
In the function `fit_lanes()` I applied the method of blind search. I divided the image into multiple horizontal windows and calculated the positions of the left and right peaks (== left and right lanes) in the histogram accordingly.
In the follow-on function `draw_polynomial()` I drew the polynomials and rectangles on the binary and warped images. 

##### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.
In the function `fill_lane()` I first set the conversion of pixels to meters in the x- and y-directions as 
```
ymeter = 30./720
xmeter = 3.7/700
```

Then I calculate the radius of the curvature

![alt text][equation_radius_curvature]

Then use the binary image and warp the image back into the regular point-of-view and finally annotate the image with the radius and vehicle position.

##### 6. Provide an example image of your plotted back down onto the road such that the lane area is identified clearly.
And here is a result of a mapped lane area:

![alt text][lane_area_image]

### C. Pipeline (video)
##### 1. Provide a link to your final video output. Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a link to my final [**video result**][video result]: 

Here's a link to my result on the [**challenge video**][challenge result]: 

Unfortunately, the lines on **harder challenge video** were too wobbly. But I would like to take that opportunity and continue it in the discussion below.

### D. Discussion
##### 1. Briefly discuss any problems / issues you faced in your implementation of this project. Where will your pipeline likely fail? What could you do to make it more robust?
Similar to our very first project about "Lane Lines", this advanced approach for finding lane lines works reasonably well in daytime when the streets are lit appropriately. However, when it comes to dark shadows or extremely bright spots on the street like in the _harder challenge video_ my pipeline fails. That is probably due to the fact that I only use two color channel gradients and not Sobel gradients or other means of identifying the pixels.

Also I found that the polynomial will be confused (see _challenge result video_) when there is another line next to the lane, that should actually not be there (such as a road filling). 

But probably the hardest will be if there are no lane lines at all. I was talking to my friend and she said that in her country lane lines aren't well noticable on the street and sometimes missing, but I guess that's beyond this project.

So to sum it up, the possible sources of error in my pipeline include:
- only LUV and Lab color space, not Sobel gradient
- eyeballing a rectangle for a perspective transform might also be a source of error

It was a lot of fun and I still have room to improve my advanced lane line pipeline program.


[docker installation]: 				https://github.com/udacity/CarND-Term1-Starter-Kit/blob/master/doc/configure_via_docker.md

[chessboard_image_corners]: 	./output_images/chessboard.png "Chessboard Corners"
[undistorted_image]: 					./output_images/undistorted.png "Undistroted Image"
[binary_image]: 							./output_images/binary.png "Binary Image"
[warped_straight_image]: 			./output_images/warped_straight.png "Warped Straight Image"
[warped_curved_image]: 				./output_images/warped_curved.png "Warped Curved Image"
[equation_radius_curvature]: 	./output_images/equation.png "Radius of Curvature Equation"
[lane_area_image]: 						./output_images/lane_area.png "Lane Area Image"
[video result]: 							./output_videos/result.mp4
[challenge result]: 					./output_videos/challenge_result.mp4
