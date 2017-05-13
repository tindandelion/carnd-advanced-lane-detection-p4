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
[video1]: ./project_video.mp4 "Video"

## Submitted files

The submission includes the following files:

* `advanced-lane-detection.ipynb` is a Jupiter notebook with the project code;
* `output_images/project_video.mp4` is a video output produced by the pipeline;
* `README.md` is a report summarizing the results

## Pipeline description

As a general approach, I decided to split the pipeline into a series of steps,
each doing a specific transformation of the input image and them, if provided,
passing the result to the next stage in the processing pipeline. This structure
allowed me to work on the pipeline incrementally, adding more processing steps
as the project progressed. 

The base class for a processing stage is `PipelineStage`, defined in cell #2.
Other steps inherit from this class and override its `process()` method to do
specific work on the input data.

### Camera calibration

The camera was calibrated on the chessboard images, provided with the
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

# Color and gradient transform

I found that in order to achieve reliable road line detection, it's best to
convert the original RGB image into HSV (Hue-Saturation-Value) space and then
use the V channel as a source. V channel provides best contrast for both yellow
and white lane lines under different lighting conditions:

![V channel extracted][image3]

The code for V channel extraction is provided in cell #6. 

Having the V channel extracted, I apply the following transforms separately:

* I apply color thresholding to remove irrelevant details;
* Sobel magnitude thresholding is used to detect the value gradient changes.

Both results are then combined with logical OR operartion to produce a resulting
binary image:

![Value threshold applied][image4]
![Sobel magnitude thesholding][image5]
![Resulting image][image6]

The code for these tranforms is located in cells #7-#9 in the notebook.



#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes a function called `warper()`, which appears in lines 1 through 8 in the file `example.py` (output_images/examples/example.py) (or, for example, in the 3rd code cell of the IPython notebook).  The `warper()` function takes as inputs an image (`img`), as well as source (`src`) and destination (`dst`) points.  I chose the hardcode the source and destination points in the following manner:

```python
src = np.float32(
    [[(img_size[0] / 2) - 55, img_size[1] / 2 + 100],
    [((img_size[0] / 6) - 10), img_size[1]],
    [(img_size[0] * 5 / 6) + 60, img_size[1]],
    [(img_size[0] / 2 + 55), img_size[1] / 2 + 100]])
dst = np.float32(
    [[(img_size[0] / 4), 0],
    [(img_size[0] / 4), img_size[1]],
    [(img_size[0] * 3 / 4), img_size[1]],
    [(img_size[0] * 3 / 4), 0]])
```

This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 585, 460      | 320, 0        | 
| 203, 720      | 320, 720      |
| 1127, 720     | 960, 720      |
| 695, 460      | 960, 0        |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![alt text][image4]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

Then I did some other stuff and fit my lane lines with a 2nd order polynomial kinda like this:

![alt text][image5]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

I did this in lines # through # in my code in `my_other_file.py`

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in lines # through # in my code in `yet_another_file.py` in the function `map_lane()`.  Here is an example of my result on a test image:

![alt text][image6]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./project_video.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

Here I'll talk about the approach I took, what techniques I used, what worked and why, where the pipeline might fail and how I might improve it if I were going to pursue this project further.  
