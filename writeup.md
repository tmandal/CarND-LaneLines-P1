# Finding Lane Lines on the Road

The goals / steps of this project are the following
* Develop an image pipeline to detect left and right lanes on the road from an image.
* Apply this pipeline on each frame of a video and project detected lanes in the video continuously.

### 1. Image pipeline description

* The color image is converted to grayscale image using provided grayscale function.
* Then, the grayscale image is blurred by applying a gaussian noise kernel of size 5x5.
* The canny edges of this blurred grayscale image are now determined with (50, 150) thresholds.
* The detected edges are masked with a triangle shaped region to exclude the regions where lanes won't possibly be found.
* The resultant edges are processed using hough line detector with its tuned-up parameters to capture hough lines.
* Finally, hough lines are clustered, filtered and extrapolated in order to identify left and right lanes.

For implementation, hough_lines function is not used. Instead, the native cv2.HoughLinesP(...) is invoked to generate hough line segments. Then a new function, *identify_lanes* is called to generate two line segments for left and right lane. These two line segments are finally fed into draw_lines function to project left and right lanes into the input image. The new function, *identify_lanes*, implements lane detection algorithm from hough line segments.

The lane detection algorithm can be described in the following steps.
* The slope, intercept and length of each hough line segment are computed. Most of horizontal line segments are rejected as part of filtering in this step.
* The above selected line segments are sorted in descending order based on their highest y coordinate so that lines can be scanned from bottom-up according to their starting endpoint.
* The line segments are grouped naively in clusters such that line segments belonging to same cluster are within tolerable distance from cluster's centroid and their slopes are within small deviations from cluster's average slope weighted by lengths of its line segments.
* A straight line is fitted for all line segments belonging to each cluster. Initial experiments showed that a polynomial fit was not a good option in this case as its fit deviated from the lane oddly when projected on input image. An average of slopes and intersects of the line segments weighted by their lengths, which is also their cluster centroid, is a good representation for fitting a line for a cluster. Additional cluster filtering is done to reject any cluster that does not have any line segment extending into bottom 33% of the image or that has average slope less than 0.4.
* All remaining clusters are finally examined to pick up closer lanes towards middle of the bottom of the image with higher total line segment lengths. Thus left and right lanes are typically found.
* An option (*identify_lanes.use_history*) is implemented to use lane lines from previous image in case the above algorithm cannot determine left or right lanes.

Here are examples of images captured at each step of this lane detection pipeline.

![alt text][image_gray]
![alt text][image_blur_gray]
![alt text][image_canny_edges]
![alt text][image_hough_lines]
![alt text][image_lane_detection]


### 2. Identify potential shortcomings with your current pipeline

While working on detecting lanes in challenge.mp4 video, I found out that the left yellow lane was impossible to detect using the above algorithm in the portion of the road where yellow marked lane on off-white road was almost blinded by bright sunlight. Canny edges and hough lines could not appropriately be detected with their parameters tweaked. Below is a capture of such an image. 

![alt text][image_undetected_lane]

The lane detection algorithm takes advantage of the observations that left and right lanes are usually on either side of the camera. If both left and right lanes are somehow captured on narrow side of the image by the camera, then the above may fail to detect both lanes.

### 3. Suggest possible improvements to your pipeline

For undetected lanes, a history of previously detected lanes from earlier frame can be maintained and this historical information can be utilized to predict a lane when undetected. 

The lane detection algorithm can be tweaked to detect both lanes on one side of the camera image if a lane is not found on one side.

[image_gray]: examples/gray.png
[image_blur_gray]: examples/blur_gray.png
[image_canny_edges]: examples/canny_edges.png
[image_hough_lines]: examples/hough_lines.png
[image_lane_detection]: examples/lane_detection.png
[image_undetected_lane]: examples/undetected_lane.jpg
