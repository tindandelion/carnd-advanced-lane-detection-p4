# Advanced Lane Finding

## Project goals

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

[image1]: ./output_images/calibration_test.png "Undistorted chessboard image"
[image2]: ./output_images/undistorted.png "Undistorted road image"
[image3]: ./output_images/v_channel.png "V channel extracted"
[image4]: ./output_images/color_threshold.png "Color threshold"
[image5]: ./output_images/sobel_threshold.png "Sobel threshold"
[image6]: ./output_images/combined_threshold.png "Combined image"
[image7]: ./output_images/perspective_transform.png "Perspective transform"
[image8]: ./output_images/test3.jpg "Output image"
[video1]: ./project_video.mp4 "Video"

## Submitted files

The submission includes the following files:

* `advanced-lane-detection.ipynb` is a Jupiter notebook with the project code;
* `output_images/project_video.mp4` is a video output produced by the pipeline;
* `README.md` is a report summarizing the results

## Pipeline description

As a general approach, I decided to split the pipeline into a series of steps,
each doing a specific transformation of the input image and then, if provided,
passing the result to the next stage in the processing pipeline. This structure
allowed me to work on the pipeline incrementally, adding more processing steps
as the project progressed. 

The base class for a processing stage is `PipelineStage`, defined in the cell
`#2`.  Other steps inherit from this class and override its `process()` method
to do specific work on the input data.

### Camera calibration

The camera was calibrated on the chessboard images, supplied with the
project. The code for this step is provided in the cell #3. The class
`CorrectDistortion` represents the pipeline stage that undistorts the input
image and passes it along.

Before invoking, the class method `CorrectDistortion.calibrate()` must be called
with the chessboard images as input. This method calculates the distortion
coefficients, invoking `cv2.findChessboardCorners()` and
`cv2.calibrateCamera()`. The coefficients are stored in class variables and used
to undistort the input image during the normal processing.

The example of undistorted chessboard image is as follows:

![Undistorted calibration image][image1]

When applied to the road images, the transform produces the following output:

![Undistorted real image][image2]

### Color and gradient transform

I found that in order to achieve reliable road line detection, it's best to
convert the original RGB image into HSV (Hue-Saturation-Value) space and then
use the V channel as a source. V channel provides best contrast for both yellow
and white lane lines under different lighting conditions:

![V channel extracted][image3]

The code for V channel extraction is provided in the cell #6. 

Having the V channel extracted, I apply the following transforms separately:

* I apply color thresholding to remove irrelevant details;
* Sobel magnitude thresholding is used to detect the value gradient changes.

Both results are then combined with logical OR operartion to produce a resulting
binary image:

![Value threshold applied][image4]
![Sobel magnitude thesholding][image5]
![Resulting image][image6]

The code for these tranforms is located in cells #7-#9 in the notebook.

### Perspective transform

The code for the perspective transform is located in the cell #10 in the
notebook. I use hardcoded source and destination point coordinates that I picked
by hand using one of the images of a straight road segment. The points are as
follows:

|      Source | Destination |
|------------:|------------:|
| (281, 668)  | (281,720)   |
| (582, 460)  | (281, 0)    |
| (703, 460)  | (1023, 0)   |
| (1023, 668) | (1023, 720) |

I verified that my perspective transform works as expected by inspecting the
transformed image and checking that the road lines appear parallel in the warped
image:

![Perspective transform][image7]

### Lane lines identification

To identify lane lines in an image I used the convolutional approach, described
in the lectures. The code for this processing step is located in the cell #12 in
the notebook. There are 2 main classes that do the work of lane detection and
approximation, `Convolver` and `LaneTracker`, described below.

The `Convolver` class is responsibe for finding the lane line points in
individual images. To do that, I split the image into 9 horizontal layers, sum
up all the pixels by Y axis to obtain the 1-dimensional array, and then apply a
convolution operation with the 50-pixel window to maximize the effect of "hot
pixels". Then, I identify the peaks in the resulting convolved signal,
separately for left and right lane boundaries. To reduce the noise, I only
search for the peaks in the 200-pixel window, the center of which is obtained
from the previously found peaks.

The X-coordinates for left and right lane lines are stored in the instance of
`LaneTracker` class, which keeps track of line points across approx. 10 last
frames. This class is also responsible for fitting the 2nd order polynomial that
approximates the entire lane. Instead of approximating left and right lane
boundaries separately, I decided to track the middle line, given that the width
of the lane is constant (the width value is found during the initialization of
the `Convolver` instance). This approach allows for a smoother approximation,
because both left and right boundary points contribute collectively to the end
result.

### Lane curvature and vehicle position

As soon as I have the coefficients of the polynomial that approximates the lane,
I can calculate the approximate curvature of the lane and the position of the
vehicle with respect to the center. This is done as a part of the final stage in
the processing pipeline, represented by the class `VisualizeLane` (cell
#12). The value in pixels is converted to metric values using the coefficients
`ym_per_pix` and `xm_per_pix`, respectively. The code for these calculations is
located in methods `curvature()` and `off_center()` of the `VisualizeLane`
class.

### Lane visualization

Also in the final stage, I produce an image of the approximated lane for each
frame, which is then unwarped with the reverse perspective transform and laid
over the source image, to produce the result. The entire pipeline is created in
a function `line_detector()` in the cell #13, that combines all the steps
described above. 

Here is the example of the single output image, produced by the pipeline: 

![Single output image][image8]

The green stripe represents the approximated lane. In addition, the line
points, extracted from the source image, are also displayed with red and blue. 

### Video processing

Here is a link to the [output video](./output_images/project_video.mp4).

## Discussion

The pipeline I've come up with performs reasonably well on the project
video. However, it's performance is not ideal on the challenge videos that have
worse conditions. I think that the following iprovements can be made to achieve
better lane detection:

1. Use additional source image transformations to produce better input for the
line detector (enhance image contrast, use gradient angle threshold, use
multiple color channels);
2. Use heuristics to analyze the approximated lane lines for individual frames
(reject abnormally curved lines, lines highly off-center);
3. When tracking line points, keep track of points for each y-coordinate
separately and reject individual points that vary too much. 
4. Apply machine learning image recognition models to detect lane lines and
filter out the noise.



