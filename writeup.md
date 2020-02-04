# **Finding Lane Lines on the Road**


---

### **Finding Lane Lines on the Road**

**Goals:**
* Create a computer vision pipeline to find lane lines on the road
* Reflect on the pipeline in this report


[//]: # (Image References)

[original]: ./report_images/00-original.jpg "Original Image"
[gray]: ./report_images/01-gray.jpg "Grayscale Image"
[canny]: ./report_images/02-canny.jpg "Canny Edge Detection"
[roi]: ./report_images/03-roi.jpg "Region of Interest"
[hough]: ./report_images/04-hough_raw.jpg "Hough Line Segments"
[lanes]: ./report_images/05-hough.jpg "Extrapolated Lane Lines"
[result]: ./report_images/06-result.jpg "Final Result"

---

### Reflection


### 1. Description of Pipeline
The pipeline is broken down into the following steps:
1. **Raw input**
![alt text][original]
1. **Grayscale conversion** - To prepare for the canny edge detection algorithm, the
image is converted to gray scale, which means that color information is lost.
Only brightness information is retained.
![alt text][gray]
1. **Canny edge detection** - The canny edge detection algorithm works by taking
the derivative of a grayscale image and analyzing the change in brightness from
pixel to pixel. Doing so allows detection of "edges", which are high contrast
areas in the image.
![alt text][canny]
1. **Mask region of interest** - A four-sided polygon is used to mask the image.
This is a straightforward way to mitigate false positives in after the Hough
transform step by filtering out regions of the image that will generally never
contain lane lines. This is possible due to the fact that the camera is affixed
to the frame of the vehicle.
![alt text][roi]
1. **Hough Transform** - The Hough transform is then used to find lines in the
image. In hough space, the intersection between curves represents points on the
same line, thus the number of intersections that occur within discrete 2D cells
within Hough space can be used as an indicator of a line.
![alt text][hough]
1. **Line Segment Filtering, Averaging, and Extrapolation**
In this final step, the line segments are filtered based on their slope and position.
Line segments are either rejected, considered part of the left lane, or considered part
of the right lane. Some reasonable thresholds for these filters were found empirically.
For each left and right lane set, the average of their slopes and
positions are calculated. Then a single segment for each lane is created by extending
the line from the bottom of the image to the top of the region of interest.
![alt text][lanes]
1. **Overlay Lanes on Input**
![alt text][result]


### 2. Shortcomings of the pipeline

1. **No frame-to-frame averaging** - The current pipeline does not have state that
carries over from one frame to the next. This means that there is no smoothing
or averaging of the lane lines. The detected lanes are derived only from the
current input video frame.
1. **Over-dependence on position** - To mitigate false positives from shadows and other
high-contrast noise, the algorithm uses position to reject line segments. For
example, a line segment that has the appropriate slope for a right lane line but
exists in the far left side of the image. The downside is that as the vehicle
begins to cross over from one lane to the next, it will struggle to detect the
lanes as they cross over the center of the vehicle.
1. **Over-dependence on slope** - Similar to the point about position above, the
slope detection logic may filter out the real lane line as it crosses over the
center of the vehicle.

### 3. Possible improvements
One possible improvement would be to use a low pass filter to smooth the output
from one frame to the next.

Another possible improvement would be to output a confidence value with each
extrapolated line. The confidence values can be used as weights when filtering
the outputs. The confidence value can be calculated based on the quality of the
line segments from the Hough transform.

Lastly, another improvement would be to spend more time and effort tuning and
validating the parameter set across a large set of input data. As mentioned in
the shortcomings section above, the over-dependence on slope and position of the lines can go
away with better signal processing earlier in the pipeline (i.e. blur + canny)
