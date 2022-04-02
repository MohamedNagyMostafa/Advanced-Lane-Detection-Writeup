# Advanced Lane Detection

## Abstract

The detection lane project consists of ordered steps as follows:

* Compute the camera calibration matrix and distortion coefficients given a set of chessboard images.
* Using distortion coefficients to correct the distortion of the input images.
* Apply color a particular color transforms and gradients in order to obtain a binary image exposes both white and yellow lane lines under almost expected scenarios such as object shadows, bright roads, and damaged roads.
* Next, the perspective transform will be applied to the binary image to obtain the bird's Eye view perspective over the road.
* Detect pixels that are most likely forming the right and left lane lines.
* Draw the area in-between the detected lane lines on a black binary image.
* Measure the curvature of the lane and vehicle position to the center.
* Ultimately, reverse the perspective transform to obtain the original image perspective.
* Combine the output with the original image and visualize the result.

## Steps

### Camera Calibration, Camera Matrix & Distortion Parameters

At the first step, the camera matrix and distortion parameters should be determined in order to have a real-world view of the objects and correct the drops of the transforming process (3D -> 2D) such as the real size of the surrounded vehicles, real lane curvature, and size. To determine the distortion parameters, we should pick up points of an object in the real-world and its points from an image from the camera and calculate the difference in shape.  To encounter that, this process is done by taking a picture of known shapes like a chessboard by various angles and positions, and then the distortion parameters will be computable. By applying these procedures over various chessboard images as well as selecting distortion parameters and camera matrix, the parameters are tested on a chessboard image as shown below.

**Test on a chessboard picture.**

![Figure_1](https://user-images.githubusercontent.com/20774864/102691443-45ef7700-4215-11eb-8e75-92ff6e25546e.png)

**Test on a picture of the road.**

![Figure_1](https://user-images.githubusercontent.com/20774864/102691911-9e744380-4218-11eb-8050-9ca7a872275c.png)

### Feature extraction, Image Gradient & Color Transforms

Extracting the White and Yellow lane lines subsequent to the Camera Calibration step. The output image goes through filters to extract both White and Yellow lane lines from the image, these filters are accurately picked up after a bunch of testing procedures under various road and light states. The testing shows that the yellow lane is detectable at almost light & road conditions in the saturation channel, and the White lane, on the other hand, are detectable in the Blue channel.


![Figure_1](https://user-images.githubusercontent.com/20774864/102692357-70dcc980-421b-11eb-94e4-d0ace407ef0b.png)

In order to eliminate the unwanted features and the objects besides emphasizing the desired features, lane lines, a Gaussian filter is applied on both channels with kernel size (11x11) to tiny distortions on the road. Afterward, the gradient has been applied to demonstrate and sharp the lane lines to have an accurate result as shown below:-


![Figure_1](https://user-images.githubusercontent.com/20774864/102692784-5526f280-421e-11eb-8dd2-b6984154e2ac.png)

Even though the previous outcome shows the lane lines sharply, the Hue channel is used to erase the background distortions. In fact, I noticed that the Hue channel shows the lane lines in black (low pixel values), and I consequently used this advantage to block the unwanted and distorted pixels in the road in addition to the surrounding objects.

![Figure_1](https://user-images.githubusercontent.com/20774864/102695639-4f86d800-4231-11eb-84ec-23780118cfd5.png)


### Perspective Transform, Bird's Eye View

At his phase, the perspective of the road changes into a bird's Eye view that makes lane detection easier. As shown below, the boundaries of this perspective are assigned to be:
* `Top_Left_Point`: 36% of the x-axis , 37% of the y-axis
* `Top_Right_Point`: 34 % of the x-axis , 37% of the y-axis
* `Bottom_Left_Point` and `Bottom_Right_Point`: 6% of the x_axis

![Figure_1](https://user-images.githubusercontent.com/20774864/102695948-6b8b7900-4233-11eb-9139-20dd0bb0a93f.png)

### Algorithm of Lane Line Detection

Here is the black box of the project. This section will be separated into two phases (algorithms); the first phase's goal is to apply an accurate detection for both left and right lane lines; the second phase's goal is to keep these detected two lines, from the first phase, stick with the new lane lines for each frame in the video.

#### First Phase Algorithm (Window Algorithm)

The main idea of the algorithm is to draw two second-degree polynomials corresponding to each lane line. Thus, two separated windows go through the two-lane lines to pick up their pixels and draw a polynomial that fits them. In this project, these two windows move ten steps from the bottom to the top of the image, searching for the lane line pixels. The algorithm could be summarized in these bullet points:

* In steady, the location of the two windows are determined through the position of the two picks on a histogram presents the pixels' values frequency over the x-axis for the half bottom of the image.

![Figure_1](https://user-images.githubusercontent.com/20774864/102696942-c96f8f00-423a-11eb-9dfd-d3220fd96cf9.png)

* After locating the initial position of the windows, pixels that have a value greater than 0 and located inside the window will be considered If the count of the pixels that contain values >0 exceeds 200 pixels. Otherwise, these pixels will not be considered. In case of having more than 200 pixels, the location of the pixels will be stored and a polynomial will be drawn to fit these pixels. Afterward, the next window will be located at point X where X is the top intersection of the drawn polynomial extension with the current window, however, there are some restrictions under particular scenarios:
    * If one of the two windows (The right window as an example) doesn't have non-zero pixels less than 200 pixels and the other one (The left window) does, the change (dx) in the new window location of the left window will be applied on the other window ( the right window) to have the same rate of change. This follows the fact that the two lines have the same direction. 
    * If the new X position for windows is located out of the image boundary, the window will solely move up with the last value of X positon.

The below figure demonstrate the location of each window at both sides and how the polynomial gets fit at each step. This algorithm works **only once** at the beginning of the detection.

![Figure_1](https://user-images.githubusercontent.com/20774864/102696701-c96e8f80-4238-11eb-9945-7fe331ed9b28.png)

#### Second Phase Algorithm (Stick To the Lane Algorithm)

After locating the lane lines in the road by the first algorithm, this algorithm keeps the drawn polynomials at each lane line fit the new frame that comes from the camera.  When a new frame comes, the surrounded points located inside margin `25` of the left and right side of the polynomial will be stored and a new polynomial will be drawn up to these new points. This algorithm comes from the fact that frames come from the camera in a tiny frame of time, which means changes of the lane line will be small and located on the top of the new frame. However, this new polynomial should have specific criteria to be accepted, otherwise, the previous one will be kept. The criteria are:
    
  * The new polynomial should fit new non-zero pixels that didn't be covered by the previous polynomial.
    
  * The polynomial curvature in addition to the distance between the top/mid/bottom points should not exceed a specific threshold

![image](https://user-images.githubusercontent.com/20774864/102698101-8f56bb00-4243-11eb-94f4-45b013275b98.png)

The figure below shows the output of the algorithm, the pixels inside the margin is colored in yellow, and the red and blue lines present the polynomial of the previous frame. The new polynomials are colored in green. Each new polynomial will go through the previous criteria if the polynomial satisfies them, It will be colored in blue or red up to which lane it indicates. Otherwise, It will remain green. 

![Figure_1](https://user-images.githubusercontent.com/20774864/102698246-ac3fbe00-4244-11eb-9ae5-3518ae6d7ce9.png)

### Measure The Curvature and The Vehicle Position to The Center

After the determination and locating of the left and right lane lines, their curvature in addition to the vehicle position since the camera is located on the top middle of the vehicle. First of all, the meter per pixel should be calculated on both the x and y axes, then the length factor should also be computed by calculating the length of a line before and after the image perspective transform. In order to compute the length factor, I did calculate the length of the bottom line of the perspective transform on the original image `Original Length` and the length of the same line after the perspective transform `Imaginary Length`. Then, the real length became computable using the following equation: 

`Real Length of Line X = (Length of Line X In the Perspective Transform * Original Length)/Imaginary Length `.

Furthermore, the vehicle position is computed by comparing the center of the image, since the camera is positioned in the center of the car, to the center of the green area between the two-lane lines. 
`Diff Position = Green Area Center - Vehicle Center`
* If the `Diff Position` value is positive that means the vehicle is left of the center by `Diff Position` meters.
* If the `Diff Position` value is negative that means the vehicle is right of the center by `Diff Position` meters.
* If the `Diff Position` value is 0 that means the vehicle is exactly in the center.

### Reverse The Perspective Transform

The perspective transform will be reversed to obtain the original result.

## Videos
Video 1: https://youtu.be/02kA-NbHyns

Video 2: https://youtu.be/IIkoIGzlVr8
