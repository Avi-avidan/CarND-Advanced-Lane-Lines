## Advanced Lane Finding

The goals / steps of this project are the following:

* Compute the camera calibration matrix and distortion coefficients given a set of chessboard images.
* Apply a distortion correction to raw images.
* Use color transforms, gradients, etc., to create a thresholded binary image.
* Apply a perspective transform to rectify binary image ("birds-eye view").
* Detect lane pixels and fit to find the lane boundary.
* Determine the curvature of the lane and vehicle position with respect to center.
* Warp the detected lane boundaries back onto the original image.
* Output visual display of the lane boundaries and numerical estimation of lane curvature and vehicle position.

[//]: # (Image References)

[image1]: ./output_images/undist.png "correcting for distortions" 
[image2]: ./output_images/undist_and_warpped.png "undistortion and warpping"
[image3]: ./output_images/perspective.png "perpective"
[image4]: ./output_images/pipeline1.png "pipeline 1"
[image5]: ./output_images/pipeline2.png "pipeline 2"
[image6]: ./output_images/final.png "final result"


### camera calibration
camera calibration was carried using images in the `camera_cal` folder, as seen in file 'cam_calibration.ipynb'.
major steps for calibration:
- images were read (cv2.imread) and converted to grayscale (cv2.cvtColor)
- Chessboard Corners were found (9x6 grid)
- camera matrix, distortion coefficients, rotation and translation vectors were calculated (cv2.calibrateCamera)
- calibration parameters were pickled and saved to file for further use

![alt text][image1]

![alt text][image2]


### finding perspective
pixels per meter in both x and y dimentions were calculated, as seen in file 'find perspective.ipynb'.
major steps:
- images were read (cv2.imread) and converted to grayscale (cv2.cvtColor)
- images were read un-distorted (cv2.undistort)
- images were warpped to get top view (cv2.warpPerspective)
- lane lines were detected using 'l' channel (after converting to HLS and thresholding above 130)
- lane lines detected (cv2.moments) for each half of the image seperately (right and left sides)
- distance between in pixels divided by 12 foot (approximated distance between lanes) * 3.28 meters per foot
- calculations were pickled and saved to file for further use

![alt text][image3]

### pipline
each frame was processed using these steps:

1. 'enhance_lines' function carry this major steps:
  - udistort, convert to HLS and smoothed (cv2.medianBlur)
  - 'l' channel derivative in x direction was calculated (cv2.Sobel) and scaled to uint8
  - binary image was created thresholds for 'h' & 's' channels and the gradient for 'l' channel

2. 'apply_mask' function carry this major steps:
  - returning the image only inside a pre-defined region of interest (ROI) (cv2.bitwise_and)

3. 'warp' function carry this major steps:
  - images were warpped to get top view (cv2.warpPerspective)

4. 'find_lanes' function carry this major steps:
  - indentify base for each lane line by getting a pick (np.argmax) of the histogram (np.sum) for the right & left side of the     image seperately
  - using a rolling window (each 1/9 of image hight), for each half of the image, nonezero values were found. if number of         nonezero indeces were over threshold (50), mean of indeces was set as the next window mid point for search.
  - a second order polynomials were fitted (np.polyfit)
  - new polynomials were fitted to x,y in world space in order to calculate radius of the road curve
  - polynomials and curves (for each lane line) were added to the Line Class (one object for each side, left and right)

5. 'Line.get_avgs' method carry this major steps:
  - returns the average line fit over previous 5 frames
  - returns the average curve calculated over previous 4 frames
  
6. 'get_line_points' function carry this major steps:
  - returns the line points matching the polynmial values fitted
  
7. 'add_lines' function carry this major steps:
  - draws the found lane lines (top view) and unwarpping them back to front view 
  
8. lane line are drawn on each frame (cv2.addWeighted)
  
9. 'get_car_shift' function carry this major steps:
  - calcultes car shift from center of the lane
  
10.'get_curve' function carry this major steps:
  - calcultes average road curve

11.'add_text_to_image' function carry this major steps:
  - adds text to the frame (cv2.putText)


![alt text][image4]

![alt text][image5]

![alt text][image6]


Here's a [link to my video result](./project_video.mp4) 
