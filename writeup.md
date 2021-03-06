# **Finding Lane Lines on the Road** 

## Writeup ouline

### 1. Project rubric
### 2. Reflection (my pipline)
### 3. Future work

---

### **1. Finding Lane Lines on the Road**

The goals / steps of this project are the following:
* Make a pipeline that finds lane lines on the road 
  - Does the pipeline for line identification take road images from a video as input and return an annotated video stream as output? (CHECK)
  - Has a pipeline been implemented that uses the helper functions and / or other code to roughly identify the left and right lane lines with either line segments or solid lines? (example solution included in the repository output: raw-lines-example.mp4) (CHECK)
  - Have detected line segments been filtered / averaged / extrapolated to map out the full extent of the left and right lane boundaries? (example solution included in the repository: P1_example.mp4) (CHECK)


[//]: # (Image References)

[image1]: ./examples/grayscale.jpg "Grayscale"

---

### **2. Reflection**

#### 1. My pipeline and understandings

Generate pipline can be described as: Input(RGB) -> Gray scale image -> Blur(denoise) -> Edge(binary) detection -> Region of interest -> Hough transform -> Draw lines 

Input(RGB) -> Gray scale image:
  - This part is easy to understand, instead of dealing with color image, dealing with only one channel image can save time
  
Gray scale image -> Blur(denoise): ([OpenCV Document](https://docs.opencv.org/3.1.0/d4/d13/tutorial_py_filtering.html))
  - Using Gaussian filter with kernel size 5. 
  
Blur(denoise) -> Edge(binary) detection
  - Using Canny detector (uint8 type result is need for Hough transform, Canny provides binary result)
  - Canny detection includes: ([OpenCV Document](https://opencv-python-tutroals.readthedocs.io/en/latest/py_tutorials/py_imgproc/py_canny/py_canny.html))
    - Smooth (did)
    - Compute gradient of using any of the gradient operators Sobel or Prewitt
    - Extract edge points: Non-maximum suppression. Looking for local maximum based on gradient magnitude and direcyion
    - Hysteresis Thresholding (gradient value): non-edge if < low_threshold, sure-edge if > high_threshold, pixels with gradient in between check if directly connect to edge

Edge(binary) detection -> Region of interest:
  - Put this step here is to decrease the computational cost for Hough transform. 
  - Draw ROI in image is helpful to visulaize the selected area
  
Region of interest -> Hough transform: ([OpenCV Document](https://opencv-python-tutroals.readthedocs.io/en/latest/py_tutorials/py_imgproc/py_houghlines/py_houghlines.html))
  - trade off between speed and resolution (angle resolution and rho resolution)
  - trade off between finding more edges and preserve main edge (min_length, min_votes, max_gap...)
  
Hough transform -> Draw lines:
  - Just draw it!


#### 2. As part of the description, explain how you modified the draw_lines() function.

Loop in all frames:

for single frame:

    for a pair vertices (x1,y1) (x2,y2):
  
    1. Calculate slop: y2-y1/x2-x1
    
    2. Separate detected line into two catagories: left lane and right lane, slop > threshold and slop < threshold
    
    3. Calculate the length and check if this line has the longest length. If so, record it as the main line in this frame
  
Now for single frame, it should has two main lines, left and right. Calculate the slops and intersections. Since we want to draw lines starting from bottom and ending at middle of the image, so we can manuuly define y1_left_new, y2_left_new, y1_right_new and y2_right_new. Finally, we could using slops and intersections to find x. 


#### 3. Identify potential shortcomings with your current pipeline


1. One potential shortcoming would be that the lines are shivering a little bit.

2. Another shortcoming could be in the challenge video, sometimes we detect wrong lanes. This happens at
 - There are tree's shadow which may create edges and at the same time if the dash lanes are two short, my draw-line algorithm will find the main line towards to other directions.
 - The edge detection works well when there are black road, however in the challenge video, there is a part that the road is very bright which lead to very small gradient value between left yellow lane and road. To detect it, I have to decrease the low_threshold and also high_threshold in Canny, but at the same time you will also find a lot noise lines. 

---

### **3. Future work**

1. A possible improvement for question 1 above would be to compare lines with previous frames. If there is no big change (threshold), no need to redraw the lines for each frame.

2. Another potential improvement for question 2 could be detecting line in color space instead of in gray scale image
