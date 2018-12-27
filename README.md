## Project report

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

[image1]: ./examples/undistorted.png "Undistorted"
[image2]: ./examples/sample_undistort.png "Calibration"
[image3]: ./examples/masked_image.png "Mask"
[image4]: ./examples/colour_spaces.png "color transforms and gradients"
[image5]: ./examples/perspective.png "Birds-eye view"
[image5]: ./examples/lane_finding_poly_fit.jpg "Fit Visual"
[image7]: ./examples/final_image.png "Final image"
[video1]: ./project_video.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

---

## README

The code for implementing entire project is contained in a IPython notebook located in "./examples/example.ipynb". Details are provided below. 

### Camera Calibration

#### 1. Perform camera calibration. 

Steps followed are as follows.
* we use chessboard images to calibrate our camera. Multiple images are used to obtain information required to overcome camera distortion. 
* Assuming the real world coordinates of chessboard coners are known (*imgpoints*), and detect the same from images (*objpoints*) using openCV function, findChessboardCorners. Using this information, we compute distortion coefficients and camera matrix with the help of calibrateCamera function in openCV. 
* Original and undistorted images are shown below. Observe that, the curved lines in original image are due to distortion and are corrected in image on right.

![alt text][image1]

### Pipeline (single images)

#### 1.Sample distortion corrected image

* we pick a sample test image and undistort it using camera matrix and distortion coefficients computed in camera calibration section.

* undistorted image is denoted as **image_undistort** in our code.

![alt text][image2]

#### 2. Masking of undistorted image

* Next, I defined a portion of the image that could most likely contain all the lane information needed and process only this part of the image, thereby avoiding all the background inforamtion. 

```python
vertices = np.array([[(100,680),
                      (575, 400), 
                      (700, 400), 
                      (1270,680)]], dtype=np.int32)

```
* This masked output is saved as **image** in our code. Further processing of lane detection is performed on this masked image.

![alt text][image3]
#### 3.Applying colour and gradient threasholds to obtain lane images

* Fundamental goal here is to identify white and yellow lane lines. After careful reading of HLS and LAB colour spaces, I am of the opinion that LAB coulour space is sufficient for this purpose. 
    * We can use Hue for detecting Yellow colour alone. Yellow is a combination of Red(Hue = 0 degree) and Green( Hue = 120 degree). Therefore an angular dimension of 30 degree to 90 degree seems to be a good threshold to start with. Based on conversion rules, for 8 bit images we can represent on 0 to 179 degrees. Hence the thresholds for H in this case to detect yellow lanes are (15, 45) and they correspond to 30 degrees to 90 degrees respectively. However, using B component in LAB coulur spaces yeilds better results. This component is better suited to capture *BLUE* and *YELLOW* colours in an image. 
    * White lines are captured using lightness value component. Note that both components are availble in LAB coulour space and hence we will use that colour space for all our lane detection. 

* It is clear in the below output that B and L components of the image in LAB colour space captures yellow and white lanes in the image.

``` python
vertices = np.array([[(120,700),
                      (575, 405), 
                      (700, 405), 
                      (1220,700)]], dtype=np.int32)

```
![alt text][image4]

#### 3. Applying perspective transformation to compute radius of curvature

* Next we applied perspective transforamtion to obtain bird's eye view of the lane. I performed a careful search for source and destination point to get a good warped image for straight lanes and used it for all images.

```python

src = np.float32(
        [[580,460],
         [710,460],
         [200,720],
         [1150,720]])
dst = np.float32(
        [[250,0],
         [1000,0],
         [250,700],
         [1000,700]])
```

This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 580, 460      | 250, 0        | 
| 710, 460      | 1100,  0      |
| 200, 720      | 250, 700      |
| 1150, 720     | 1100, 700     |

* Observerd that both lanes are parallel as shown below.

![alt text][image5]

#### 4. Detecting lane pixels and fitting a polynomial for lane markings

* Next, using the bird's eye view of the processed lane image, a polynomial is fitted for the lane lines. 

* Sliding window approach, recoginising clusters of lane pixels is followed to group pixels corresponding to left and right lanes as shown in below figure.

* Numpy function *polyfit* is used to fit a second order polynomial for both lanes. 

![alt text][image6]

#### 5.Computing Radius of curvatue and deviation from center of lane in real world

* Using the polynomial fit computed in above secition, radius of curvature of both lanes is computed for real world usage directly by using mathematical equations.

* For conversion from pixels to real space, followed below scaling values:
    - ym_per_pix = 30/260 # meters per pixel in y dimension
    - xm_per_pix = 3.7/750 # meters per pixel in x dimension

* Obtained these values using the fact that lane width is 3.7m and length of dotted lines is 10 ft and are spaced 30ft apart. 

* Further, assuming that the camera is mounted on the center of the lane in a striaght lane, I computed the position with respect to center of lane by substracting the mid point of left fit  and right fit lanes and the center of image.

#### 6. Example image of your result plotted back down onto the road such that the lane area is identified clearly.

![alt text][image7]

---

### Pipeline (video)

#### 1. Final video output.  

Here's a [link to my video result](./output_images/result.mp4)

---



