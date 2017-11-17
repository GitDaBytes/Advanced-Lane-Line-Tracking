## Writeup Template

### You can use this file as a template for your writeup if you want to submit it as a markdown file, but feel free to use some other method and submit a pdf if you prefer.

---

**Advanced Lane Finding Project**

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

[image1]: ./output_images/calibration1_undistort.png "Undistorted"
[image2]: ./output_images/test_undistort.png "Road Transformed"
[image3]: ./examples/binary_combo_example.jpg "Binary Example"
[image4]: ./output_images/road_original.png "Road Warp Points"
[image5]: ./examples/color_fit_lines.jpg "Fit Visual"
[image6]: ./examples/example_output.jpg "Output"
[video1]: ./project_video.mp4 "Video"
[image7]: ./output_images/calibration1_distort.png "Distorted"
[image8]: ./output_images/road_warped.png "Road Warped"
[image9]: ./output_images/road_unwarped.png "Road Unwarp"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README

Note: The main code for this project was compiled in the IPython notebook located at ./AdvancedLaneFind.ipynb

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the first code cell of the IPython notebook located in "./examples/example.ipynb" (or in lines # through # of the file called `some_file.py`).  

The first step is to remove and distortion that is introduced by the camera / camera lens pair. In order to do this, we take a series of images (at least approx. 20 images) of a known object on a flat plane, with that object in varying poses around the camera field of view.

The object we use is a chessboard pattern printed on a flat surface. Next, we prepare a list of known target points (x,y,z), where we set the z to zero for all points. This is our known target. Each point maps to a corner of a grid cell on a chessboard pattern. Then we loop through each of the ~20 images comparing the chessboard pattern in the image with the known truth image.

Once we have done this for each image, we can pass these values into `cv2.calibrateCamera()` to calculate camera parameters that describe the amount of distortion the camera / lens pair introduces to images it takes. We can use these parmeters to "fix"  / undistort any images that this camera now takes, thus greatly reducing error in any measurements we want to make with those images.

Below is an example of an original distorted image, and another that has been rectified / undistorted.

Original (distorted) image:

![alt text][image7]

Undistorted  / corrected image:

![alt text][image1]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

The next thing I tried was to load an image of a road scene taken with the same camera / lens pair and apply `cv2.undistort` to test the image correction on a regular photo. The result can be seen below:

![alt text][image2]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used a combination of color and gradient thresholds to generate a binary image (thresholding steps at lines # through # in `another_file.py`).  Here's an example of my output for this step.  (note: this is not actually from one of the test images)

![alt text][image3]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

In order to identify where we are in the road, and whether the road is curving ahead, we need to change the camera perspective to a top down view. With this view, if you are centered and the two lines are parallel up the image, then we know we are centered and driving on a straight road.

To change the image perspective we first take an image with as straight a road as possible, and mark four points on it. I used two points close to the car (one on each lane line), and two up ahead of the car, one on each lane line.

Next we make up four points that we would like to map the first four points to. These should be two parallel lines that run the full extent of your image from top to bottom.

We then use `getPerspectiveTransform` to build the mapping from the original viewpoint to the new viewpoint (we can also build an inverse of this mapping to move the other way too).

I built all of this into a function called: GenerateCam2TopDownTransform()

I hard coded the points to map to / from as follows (these points were obtained through a paint program):

```python
inputPoints = np.array([[601, 446], [682, 446], [1014, 660], [302,660]],np.float32)
outputPoints = np.array([[280, 0], [1000,0], [1000,720], [280,720]],np.float32)
```

This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 601, 446      | 280, 0        | 
| 682, 446      | 1000, 0       |
| 1014, 660     | 1000, 720     |
| 302, 660      | 280, 720      |

I checked all was working as expected by drawing target points on the original image, then using `warpPerspective` to make sure I got an image of two parallel road lines. Finally I warped that image back to the original driver view.

* Original image with selected points marked:
![alt text][image4]

* Image warped to top down view:
![alt text][image8]

* Top down image warped back to original driver view:
![alt text][image9]

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
