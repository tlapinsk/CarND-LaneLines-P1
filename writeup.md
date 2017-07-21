# **Finding Lane Lines on the Road** 

Goals:
1. Pull the pieces together from the quizzes and fully understand how a basic image pipeline works.
2. Gain a better understanding of how Python can be used to implement algebra.
3. Have fun!

---

## Reflection

### My pipeline

1. First I converted all of the images to grayscale. This allows us to target only the white pixels, instead of sifting through many different colored pixels.

![alt text](https://github.com/tlapinsk/CarND-LaneLines-P1/blob/master/output_images/gray_solidWhiteCurve.jpg?raw=true "Grayscale Output")

2. Next up was applying a Gaussian blur (kernal size of 5) to smooth out the image.

![alt text](https://github.com/tlapinsk/CarND-LaneLines-P1/blob/master/output_images/gaussian_solidWhiteCurve.jpg?raw=true "Gaussian Blur Output")

3. I then applied Canny edge detection, defined a 4 sided polygon, and masked the image to pull out only the regions of interest. Meaning, the white pixels on a black background.

![alt text](https://github.com/tlapinsk/CarND-LaneLines-P1/blob/master/output_images/masked_solidWhiteCurve.jpg?raw=true "Masked Output")

4. Next, I defined the Hough transform parameters for optimization purposes, and then ran it through the `hough_lines` helper function. It was then time to run it through the `weighted_img` function to display the lines on top of the road (combine images and overlay).

![alt text](https://github.com/tlapinsk/CarND-LaneLines-P1/blob/master/output_images/overlay_solidWhiteCurve.jpg?raw=true "Overlay Output")

5. Now comes the most important piece: Working through some algebra to create one straight line on either side. 

First, we need to create a few lists to store slope and intercept points. 

	m_right = []
	i_right = []
	m_left = []
	i_left = []

Next up was building on top of the for loop to grab the slope and intercept values for each line. This was achieved by setting the slope of a line formula against a comparison operator. Since the axis is flipped, positive will indicate the right hand line and negative will indicate the left hand line. 

	for line in lines:
	 	for x1,y1,x2,y2 in line:
		  if (y2-y1)/(x2-x1) > 0:
		      m = (y2-y1)/(x2-x1)
		      i = y1-(y2-y1)*x1/(x2-x1)
		      m_right.append(m)
		      i_right.append(i)
		  if (y2-y1)/(x2-x1) < 0:
		      m = (y2-y1)/(x2-x1)
		      i = y1-(y2-y1)*x1/(x2-x1)
		      m_left.append(m)
		      i_left.append(i)
    
It was then time to use the points stored in the lists to find the min/max x and y values so that we can create a line on either side.

	m_middle_right = np.median(m_right)
	i_middle_right = np.median(i_right)
	x_bottom_right = int((540-i_middle_right)/m_middle_right)
	y_bottom_right = 540
	x_top_right = int((330-i_middle_right)/m_middle_right)
	y_top_right = 330

	m_middle_left = np.median(m_left)
	i_middle_left = np.median(i_left)
	x_bottom_left = int((540-i_middle_left)/m_middle_left)
	y_bottom_left = 540
	x_top_left = int((330-i_middle_left)/m_middle_left)
	y_top_left = 330

The last step was to use the OpenCV Drawing Function (cv2.line) to plot the points onto the image. I adjusted the thickness to 7 so that the lines showed up more clearly.

	cv2.line(img,(x_bottom_right,y_bottom_right),(x_top_right,y_top_right),color,thickness)
	cv2.line(img,(x_bottom_left,y_bottom_left),(x_top_left,y_top_left),color,thickness)

6. This last step created a solid long line that ran in both the image and video pipelines

![alt text](https://github.com/tlapinsk/CarND-LaneLines-P1/blob/master/output_images/lines_solidWhiteCurve.jpg?raw=true "Final Output")

### 2. Identify potential shortcomings with your current pipeline

1. The pipeline could be further optimized to keep the lines from "shaking" in the videos.

2. I did not attempt the Challenge at the end, so my pipeline is not optimized for this.

### 3. Suggest possible improvements to your pipeline

1. It may be appealing to others if the final lines were a tad thicker.

2. Cleaner code would definitely make it more readable for others.
