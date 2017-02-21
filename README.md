# Finding Lanes on the Road
This is the submission for P1 project for Udacity Nano degeree for Self Driving Car.

The goals / steps of this project are the following:

* Make a pipeline that finds lane lines on the road
* Reflect on your work in a written report


[//]: # (Image References)

[image1]: ./examples/grayscale.jpg "Grayscale"
[image2]: ./test_images/solidYellowCurve.jpg
[image3]: ./test_images/processed/solidYellowCurve.jpg

---

### Reflection

###1. Describe your pipeline. As part of the description, explain how you modified the draw_lines() function.

My pipeline consisted of the following steps. 

1. Convert the image to Gray Scale
2. Apply a Gaussian Blur transform to smoothen out noise and remove spurious edge
3. Detect edges using canny edge detection
4. Create an image mask that serves as Region of Interest. Any pixels outside the ROI are ignored from all further processing
5. Find lines using Probabilistic Hough Line Transform on the image mask 
6. Merge masked image on top of the original image so that the detected lines are now seen on top of the original image

Here are some of the images from my pipeline. First image is the original image.
![alt text][image2]


Final image showing the detected lanes on top of the original image
![alt text][image3]

Step 5 if the pipeline was the most interesting part of the project. Although the final solution was simple, it required me to 
go back to basic line geometry to find a good solution.

In order to draw a single line on the left and right lanes, I modified the draw_lines() function as follows:

- I first had to separate each line segment from the Hough Lines to either belong to the left lane or the right lane. For this, I found the slope of each line. The sign of the slope was used to classify it as left lane or right lane. Slopes beyond a buffer envelope were ignored. This helped in filtering out segments that would make the final lines too jittery or filter out the effect of noise

- In order to get an average slope of all the line segments, I tried various approaches. Initially, I tried using a simple arithmetic average of all slopes of each lane. It worked well for the test images. But it was found to be very problematic for the videos. The dashed lines were very jittery and at times the 2 lanes would cross each other. I tried using min and max of the x & y co-ordinates of the boundbox formed after getting all line segments. This was also found to be error prone with the dashed lines. Finally, I used line polynomial line fitting of the 1st degree to find the average slope of the line for each lane. Before using Polynomial, I tried Least Square transform. It gave similar results as Polynomial transform. But I found Polynomial line to be more stable.

- To extrapolate the lane lines to fit the complete ROI, I used the slope and intercept I got from the Poly fit. Using the y co ordinates of the ROI, I found the X Co ordinates for each line using the line equation y = slope * X + intercept




###2. Identify potential shortcomings with your current pipeline


Some of the shortcomings of my current pipeline are:
- My pipeline cannot handle extreme lighting conditions. It will work only if the image has a good, even contrast
- The pipeline does not work well if the road is curving. It does not work for the challenge video
- It may have a hard time finding the lanes if the markers are faint or there is a lot of noise around the markers
- I have assumed a fixed ROI. If the car were to move across the lane or weave within the lane etc, this pipeline will not work



###3. Suggest possible improvements to your pipeline

To overcome some of the shortcomings of the pipeline, I think I can make the following improvements:
- Improve the contrast using histogram equalization or Clahe. It is likely that I may need to also play with the color improvement for detecting lanes under certain conditions
- For finding lanes for curved roads, I think doing a curve fitting over Poly fit may work better. Also, may need to either replace Hough Lines or tweak its parameters so that it can detect curves better
- Find a way to do adaptive ROI selection rather than hardcoding it
- May need to also do some amount of sharpening before edge detection
