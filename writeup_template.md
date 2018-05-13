# **Finding Lane Lines on the Road** 

## Writeup Template

### You can use this file as a template for your writeup if you want to submit it as a markdown file. But feel free to use some other method and submit a pdf if you prefer.

---

**Finding Lane Lines on the Road**

The goals / steps of this project are the following:
* Make a pipeline that finds lane lines on the road
* Reflect on your work in a written report


[//]: # (Image References)

[gray_image]: ./test_images_output/gray_image.jpg
[blurred_image]: ./test_images_output/blurred_image.jpg
[cannyed_image]: ./test_images_output/cannyed_image.jpg
[cropped_image]: ./test_images_output/cropped_image.jpg
[line_image]: ./test_images_output/line_image.jpg
[merged_image]: ./test_images_output/merged_image.jpg
[line_image_2]: ./test_images_output/line_image_2.jpg
[merged_image_2]: ./test_images_output/merged_image_2.jpg

---

### Reflection

### 1. Describe your pipeline. As part of the description, explain how you modified the draw_lines() function.

My pipeline consisted of 6 steps. 

First, I took a single image in RGB(8-bit * 3) format and then converted it to grayscale(8-bit), which is handier to deal with. The color difference is not a thing we care about, but the intensity values are the key point to solve the problem. In this case, I applied Grayscale transform function, cv2.cvtColot(), provided from openCV.
![gray_image]




Second, I applied Gaussian blurring to the grayscaled image with 3x3 kernel in order to reduce the noise and spurious gradients by averaging.
![blurred_image]




Third, I applied Canny Edge detection algorithm to the blurred image with low threshold value = 100, high threshold value = 200. In this algorithm, the pixels lower than the low threshold are rejected and the strong edge(strong gradient) pixels above the high threshold are detected. Finally, the pixels between the low and high thresholds are included once they are connected to the strong edges.
![cannyed_image]




Fourth, I cropped the image while only leaving the region where I expect the lane lines to be appeared. By using the quadrilateral region mask using the cv2.fillpoly() provided from openCV, I can filter out detected line segments in other areas of the image.
However, there are infinitely many lines that pass through each of the edge pixels, so we cannot simply test all of the lines to see if they match many of the pixels around a selected pixel. So I used a mathematical trick to transform the input in such a way that there is a single solution for each line that we want to discover.




Fifth, Hough transform with distance resolution in pixels = 6, angular resolution in radians = pi/60, minimum intersections in Hough griod = 20, minimum line length = 3, maximum line gap is applied to the image. In Hough Space, each line represents a point from Image Space, and each point represents a line from Image space.
I need to figure out more in detail about the response from the changes of each parameter though.
![line_image]





Sixth, draw lines and merge it with the original image.
With the two endpoints [x1, y1, x2, y2] of the detected line segment from Hough Transform function, I can draw the lines with each slope (y2-y1)/(x2-x1). The slope can help me decide which segments are part of the left line vs. the right line. In the image coordinate system, as the origin is in the top left corner, the slope directions will be reversed from the general thought, with negative slopes traveling upwards and positive slopes traveling downwards.
In my image, the left lane markings all have a negative slope, while all of the right lane markings have a positive slope. 
Furthermore, I can reject some lines with a slope absolute value less than 0.5, which is hardly considered as a lane line.
![merged_image]


With these algorithms above, there was no big difficulty to find out the proper lane. But for videos, at the beginning I just drew a line directly connected from end point to end point of line. But it was too sensitive to the end point, resulting in rapidly changing slopes. 
To solve the issue, I had to average and extrapolated the lines in each side separately. I accumulated each point of those sides, and then first degree polynomial y=mx+b was fit to those points using numpy.polyfit function. When the line equation was found, it was evaluated on the region of interest to define the points where the line would be shown on the image.
![merged_image_2]




### 2. Identify potential shortcomings with your current pipeline

While I got a satisfactory result on the first two videos provided by Udacity, I identified there are some issues on my current pipeline for the optional challenge video. Although I tried to figure it out hard, but it remains still quite tricky.

Overall, the challenge video has some specific features such that:
1) The road becomes very lighter at a certain point. It destroys the canny edge detector which finds the line using a grayscaled image.
2) The car is driving on a pretty much curved road. I may need to modify some parameters of Hough transform function.
3) There are some shadows due to some trees


One potential shortcoming would be what would happen when ... 

Another shortcoming could be ...


### 3. Suggest possible improvements to your pipeline

A possible improvement would be to ...

Another potential improvement could be to ...
