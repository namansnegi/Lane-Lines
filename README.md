#**Finding Lane Lines on the Road** 
[![Udacity - Self-Driving Car NanoDegree](https://s3.amazonaws.com/udacity-sdc/github/shield-carnd.svg)](http://www.udacity.com/drive)

<img src="laneLines_thirdPass.jpg" width="480" alt="Combined Image" />

## FINDING LANE LINES 
The job of an autonomous car engineer is to teach the car how to drive by itself. The first step in achieving this task is to teach the car how to perceive the world around it. Perception is the ability to organize, identify and interpret sensory data in order to represent and understand the environment. While driving, a human driver uses his eyes to see and identify things like traffic signs, lane lines, corners, other vehicles etc. and takes the necessary control action accordingly. A car does not have eyes but things like cameras and sensors can give us information from which we can derive visual perception of the world around us. This article describes the process of using camera images to identify and track different objects while driving. We start by identifying lane lines in still images and then use the same process in a video stream to track the lane lines in real time. 

### FEATURE SELECTION
Before we begin processing the data we first have identify the features that can be useful in figuring out where the lane lines are in an image. Some of the features that can be explored are colour, shape, orientation and position in the image. 

#### Colour Selection
Lane lines are generally of single colour like white or yellow. To identify lines based on colour we have to figure out how to select a specific colour in an image. Colour in case of digital images typically means that the image comprises of a stack of different images, e.g. red green and blue in case of RGB colour space. These images are sometimes referred to as colour channels and contain pixels whose value vary from 0 to 255 where 0 is the darkest possible value and 255 is the brightest. Hence pure white colour in a RGB image will have pixel value [255,255,255]. Hence combination of these three pixel values can help us in selecting any colour in an image. The figure below shows the output of the colour selection algorithm with threshold pixel values [200,200,200] i.e. any pixels below these threshold values were set to zero.    
![Test Image](https://github.com/namansnegi/Lane-Lines/blob/master/images/1.png) 
![Color Selection](https://github.com/namansnegi/Lane-Lines/blob/master/images/2.png)
#### Region of Interest
Camera images contain lot of information and sometimes we do not need all this information as it would reduce the processing speed and also sometimes detect some other objects that aren't lane lines. Hence we would like to focus only in the area where we think the actual lane lines are. We can assume that the camera that took the image is mounted in a fixed position on the car, such that the lane lines will always appear in the same general region of the image and we would like to only consider pixels in this specific region for colour selection. 
To select a particular region in the image we can manually choose variables for left, right, and apex to represent the vertices of a triangular region for colour selection, while masking everything else out. Any polygon shape can be used for this process. All pixels that lie outside the defined region are not considered in the colour selection process. The figure below shows the selected region of interest and the identified lane lines in that region.

![ROI Image](https://github.com/namansnegi/Lane-Lines/blob/master/images/3.png) 
![ROI Lines](https://github.com/namansnegi/Lane-Lines/blob/master/images/4.png)

#### Disadvantages of Using Colour Selection
Although this method of identifying lane lines is quite straightforward and easy and gives relatively good results, it is not very robust. As it happens, lane lines are not always the same colour, and even lines of the same colour under different lighting conditions (day, night, etc) may fail to be detected by this method. We would now look into some computer vision techniques that would help us in writing a more robust algorithm. Computer vision involves using algorithms to help the computer see the world like we see it, full of depth, colour, shape and meaning. There are many techniques that can be used and in this article we will look into two of these techniques

### CANNY EDGE DETECTION
The Canny Edge detector was developed by John F. Canny in 1986. It is also known as the optimal detector. The concept is to find boundaries of an object in an image. Using this method we try to find edges by tracing the pixels that follow the strongest gradient. A grayscale image consists of points of varying brightness. Gradient of an image defines how fast the pixel values change at each point and in which direction is this change more predominant.  If we take the gradient of a grayscale image we get an image where the brightness of each pixel gives the strength of the gradient at that point. Rapid change in brightness is where we find edges. The strength of an edge is defined by the difference in values of adjacent pixels (i.e. the strength of the gradient).

#### Noise Reduction
Before computing the gradient for Canny edge detection, Gaussian smoothing is applied to the image to suppress noise and spurious gradients by averaging. Gaussian filtering is done by convolving each point in the input array with a Gaussian kernel and then summing them all to produce the output array. Kernel size for Gaussian smoothing can be any odd number where a large kernel size implies averaging, or smoothing, over a larger area. 

#### Finding Intensity Gradient of the Image
Gradient of an image can be derived using Sobel kernel in both horizontal and vertical direction. 

![Sobel](https://github.com/namansnegi/Lane-Lines/blob/master/images/10.png) 

The overall magnitude and direction of the gradient can then be calculated as beow:

![Sobel Grads](https://github.com/namansnegi/Lane-Lines/blob/master/images/11.png)

Gradient direction is always perpendicular to edges. It is rounded to one of four angles representing vertical, horizontal and two diagonal directions.

#### Non-maximum Suppression
The gradient method results in thick edges. Edge-thinning algorithm known as Non-max suppression (NMS) is applied which results in one pixel wide ridges from thick edges.  Edges must be placed at the points of maxima; or rather non-maxima must be suppressed. Non maximum suppression works by finding the pixel with the maximum value in an edge. The image is scanned along the image gradient direction, and if pixels are not part of the local maxima they are set to zero. The basic idea of NMS is that: If a pixel value is not greater than its neighbourhood pixels, then the pixel is not the edge (the scalar value of the pixel is set to zero).  The result is a binary image with "thin edges".

#### Hysteresis Thresholding
This stage is used to eliminate the edges that could not be suppressed by non-maximum suppression and also to remove noise from the image. For this process we need two threshold values, low threshold and high threshold. Any edges with intensity gradient more than high threshold are categorized as strong edges and those below low threshold are considered as non-edges and hence discarded by setting their pixel value to 0. The edges in between these thresholds are considered weak edges.  Final step is to identify which of the weak edges is a real edge which is done based on their connectivity. If they are connected to strong edges then they are considered to be actual edges or else they are also discarded.

The figure below shows the grayscale image and the output of the Canny Edge detection process. 

![GrayScale Image](https://github.com/namansnegi/Lane-Lines/blob/master/images/5.png) 
![Canny Edge](https://github.com/namansnegi/Lane-Lines/blob/master/images/6.png)
 	 
### HOUGH TRANSFORM
Canny Edge detection returns edges in the form of pixels (dots) that represent an edge. Next step is to join these dots. They can be joined to represent any kind of shape. But we are interested in looking for lines. For this we adopt a model of a line to fit the assortment of dots. A straight line can be defined by the equation y=mx+c where ‘m’ and ‘c’ represent the parameters of the line. In image space, a line is plotted as x vs. y, but in 1962, Paul Hough devised a method for representing lines in parameter space, which is known as “Hough space”.  The Hough Transform is just the conversion from image space to Hough space. So, the characterization of a line in image space will be a single point at the position (m, c) in Hough space.
![Hough](https://github.com/namansnegi/Lane-Lines/blob/master/images/9.png) 	

All points on a line in image space intersect at a common point in parameter space. This common point (m, b) represents the line in image space. Unfortunately, the slope, m, is undefined when the line is vertical. To overcome this we can use polar co-ordinates. Each point in the image space now represents a sine curve in Hough space. 
To convert the Cartesian form y=mx+c with parameters ‘m’, ‘b’ )to polar form with parameters ‘ρ’ ,’θ’ we define  as the perpendicular distance of the line from the origin and is the angle the perpendicular to the line makes with the axes. 
![Derive](https://github.com/namansnegi/Lane-Lines/blob/master/images/8.png) 
y=mx+c
y=c (when x=0)
Hence c=ρ/sin⁡θ 
now put y=0
x=-ρ/(m sin⁡θ )
cos⁡〖θ=ρ/((-ρ)⁄〖m sin〗⁡θ )〗
Hence m= -cos⁡Ө/sin⁡Ө 
y= -   cos⁡θ/sin⁡θ   x+ρ/sin⁡θ 

So if line is passing below the origin, it will have a positive rho and angle less than 180. If it is going above the origin, instead of taking angle greater than 180, angle is taken less than 180, and rho is taken negative. Any vertical line will have 0 degree and horizontal lines will have 90 degree

![Canny Edge](https://github.com/namansnegi/Lane-Lines/blob/master/images/6.png) 
![Hough Transform](https://github.com/namansnegi/Lane-Lines/blob/master/images/7.png) 

Above is the output of the HoughLinesP function in OpenCV (Python). The function takes the following arguments
	image – 8-bit, single-channel binary source image. The image may be modified by the function.
	lines – Output vector of lines. Each line is represented by a 4-element vector (x1, y1, x2, y2), where  (x1, y1) and (x2, y2)  are the ending points of each detected line segment.
	rho – Distance resolution of the accumulator in pixels.
	theta – Angle resolution of the accumulator in radians.
	threshold – Accumulator threshold parameter. Only those lines are returned that get enough votes ( > threshold ).
	minLineLength – Minimum line length. Line segments shorter than that are rejected.
	maxLineGap – Maximum allowed gap between points on the same line to link them.





