# **Finding Lane Lines on the Road** 

Goals:
1. Pull the pieces together from the quizzes and fully understand how a basic image pipeline works.
2. Gain a better understanding of how Python can be used to implement simple algebra.
3. Have fun!

---

## Reflection

### My pipeline

Shoutout to other students who helped provide structure and inspiration - especially for connecting the lines using basic algebra.

1. First I converted all of the images to grayscale. This allows us to target only the white pixels, instead of sifting through many different colored pixels.

![alt text](https://github.com/tlapinsk/CarND-LaneLines-P1/blob/master/output_images/gray_solidWhiteCurve.jpg?raw=true "Grayscale Output")

2. Next up was applying a Gaussian blur (kernal size of 5) to smooth out the image.

![alt text](https://github.com/tlapinsk/CarND-LaneLines-P1/blob/master/output_images/gaussian_solidWhiteCurve.jpg?raw=true "Gaussian Blur Output")

3. I then applied Canny edge detection, defined a 4 sided polygon, and masked the image to pull out only the regions of interest. Meaning, the white pixels on a black background.

![alt text](https://github.com/tlapinsk/CarND-LaneLines-P1/blob/master/output_images/masked_solidWhiteCurve.jpg?raw=true "Masked Output")

4. Next, I defined the Hough transform parameters for optimization purposes, and then ran it through the `hough_lines` helper function. It was then time to run it through the `weighted_img` function to display the lines on top of the road (combine images and overlay).

![alt text](https://github.com/tlapinsk/CarND-LaneLines-P1/blob/master/output_images/overlay_solidWhiteCurve.jpg?raw=true "Overlay Output")

5. Now comes the most important piece: Working through some algebra to create one straight line on either side. 

First, we need to create a few lists to store left and right slope/intercept. 

	r_slope = []
	l_slope = []
	r_intercept = []
	l_intercept = []

Next up was building on top of the for loop to grab the slope and intercept values for each line. This was achieved by setting the slope of a line formula against a comparison operator. Since the axis is flipped, positive will indicate the right hand line and negative will indicate the left hand line. 

	for line in lines:
		for x1, y1, x2, y2 in line:
			if (y2 - y1)/(x2 - x1) > 0:
				r_slope.append((y2-y1)/(x2-x1))
				r_intercept.append(y1-(y2-y1)*x1/(x2-x1))
			elif (y2 - y1)/(x2 - x1) < 0:
				l_slope.append((y2-y1)/(x2-x1))
				l_intercept.append(y1-(y2-y1)*x1/(x2-x1))
			else:
				pass

To average the values from the list, I first tried taking the mean (np.mean). The video output was a bit too jumpy for my liking, so I decided to take the median instead (problem solved).

	r_median_m = np.median(r_slope)
	l_median_m = np.median(l_slope)
	r_median_i = np.median(r_intercept)
	l_median_i = np.median(l_intercept)
    
It was then time to use the median variables to create top and bottom x,y values that would feed into the cv2.line function. This was achieved with the help of two forum posts:

1. [subodh.malgonde's June 17th response](https://discussions.udacity.com/t/project-1-finding-lanes/259763/32)
2. [fernandodamasio's June 1st response](https://discussions.udacity.com/t/lane-extrapolation-help/253653/3), specifically this [YouTube video](https://www.youtube.com/watch?v=ibxvtsMOVgQ)

The pseudo code helped frame my approach and fernandodamasio's YouTube video link helped me understand how to estimate y values based on the image size. You can see below that I estimated 540 for the bottom y values and 335 for the top y values, which were then used with the equation x = (y - c)/m to find integer values. Check out the code below:

	r_bottom_y = 540
	r_bottom_x = int(round((r_bottom_y-r_median_i)/r_median_m))
	r_top_y = 335
	r_top_x = int(round((r_top_y-r_median_i)/r_median_m))
  
	l_bottom_y = 540
	l_bottom_x = int(round((l_bottom_y-l_median_i)/l_median_m))
	l_top_y = 335
	l_top_x = int(round((l_top_y-l_median_i)/l_median_m))

The last step was to use the OpenCV Drawing Function (cv2.line) to plot the points onto the image. I adjusted the thickness to 9 so that the lines showed up more clearly.

	cv2.line(img,(r_bottom_x,r_bottom_y),(r_top_x,r_top_y),color,thickness)
	cv2.line(img,(l_bottom_x,l_bottom_y),(l_top_x,l_top_y),color,thickness)

### Identify potential shortcomings with your current pipeline

1. The pipeline could be further optimized to "shake" less. Particularly near the bottom part of the lane overlays.

2. The two videos we tested were instances of ideal weather conditions. In real world scenarios, the pipeline will also need to handle different lighting and weather conditions.

### Suggest possible improvements to your pipeline

1. My pipeline is not capable of handling the challenge video.
