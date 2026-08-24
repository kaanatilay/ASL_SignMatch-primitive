
**Sign-Match ASL**(primitive)
-
Kaan Atilay 

**Overview**
-

Sign-Match ASL filter is a video-based prototype for recogniing a limited set a of ASL (American Sign Language) gestures which can be signed using only a single hand. The system uses a color-coded glove to identify each individual hand parts, normalize the complete hand and compare resulting respresentation masks against stored gesture templates.
    
The current system vocabulary is limited to: 
    
    [B, E, K W]

**Process**
-
The system consist of 4 main stages:

**1. Color Calibration**

    Before processing gestures, the system performs a short calibration using a live webcam. Each finger, the palm, and the finger-base markers are assigned HSV color thresholds based on a selected area of the glove.

    Calibration allows the system to adjust its color segmentation to different lighting and recording conditions.

**2. Hand-Part Detection**

    Each hand part is segmented from the frame using its calibrated HSV range.

    For each finger, the system:

    - Detects the largest contour associated with the finger color.
    - Performs morphology to reduce gaps and noise in the detected region.
    - Detects the finger-base marker and finds its intersection with the finger region by dilating both.
    - Uses the intersection to estimate the finger base.
    - Selects the furthest contour point from the base as the fingertip.

    The palm is detected separately, with its center calculated from the detected palm region. 

**3. Normalization**

    Detected gestures can appear at different positions and sizes within a video frame. To make template comparison more consistent, the detected hand is transformed into a standardized 400 × 400 gesture representation.

    The palm center is used as the reference position, while palm width determines the required scaling. The visible hand-part masks are then translated and scaled into the normalized representation making the complete masks easily comparable and consistent.

**4. Gesture Matching**

    The normalized gesture is compared against each stored gesture template.

    Similarity is calculated using Mean Squared Error (MSE) across the active(visible) pixels of the detected gesture and template. An additional penalty is applied when the set of visible hand parts differs from what is expected for a particular gesture.

    The gesture with the lowest similarity score is selected as the closest match. A recognition is accepted only when its score is below the configured similarity threshold and the gesture remains consistently detected across the required number of frames.

**Project Repository Structure**
-

    Sign-Match-ASL/
    |
    |_sign_match_ASL.ipynb
    |_ gesture_templates/
    |   |_ raw/
    |   |_ normalized/
    |
    |_ inputVids/
    |_ resultVids/
    |_ temp_DevImages/masks/
    |   |_ index
    |   |_ middle
    |   |_ palm
    |   |_ pinky
    |   |_ ring
    |   |_ thumb
    |_ dependencyInstaller.txt

**Runing the Project**
-

**1. Install Dependencies**

The required Python libraries can be installed using:

    pip install -r dependencyInstaller.txt

**2. Prepare the Color-Coded Hand**

The system requires a distinct color for each tracked hand part. The colors used during development were:

|Hand Part | Color
|:--|---:|
Finger base | Yellow
Thumb | 	Pink
Index finger | 	Orange
Middle finger | 	Blue
Ring finger	 | Green
Pinky finger | 	Purple
Palm | 	Black

    Different colors can be used because the HSV ranges are determined during calibration, although visually distinct colors are recommended.


**3. Run Calibration**

Run the notebook through the CALIBRATION section and use the webcam window to calibrate each hand part.

Select a hand part using its corresponding key and then left-click the colored region in the webcam view:

Key	|Calibration Target
|:-:|:--:
y |	Finger base
a |	Thumb
s | Index
d |	Middle
f |	Ring
g |	Pinky
h |	Palm
m |	Toggle calibration instructions

All hand parts and the finger-base marker should be calibrated before processing gestures.

**4. Gesture Templates**

Existing normalized templates for the supported vocabulary can be loaded directly by the recognition pipeline.

The notebook also contains sections for creating additional templates:

<u>Saving raw representations of each gesture</u> captures a gesture from the webcam.
<u>NORMALIZING THE RAW GESTURES</u> converts a captured representation into the standardized template format.

**5. Process an Input Video**

The sign_match_ASL(inputVidPath, vidNo) function processes an input video and returns the recognized gesture results.

For example:

    results, debug_results = sign_match_ASL(
        "inputVids/input_videoDemo.mp4",
        0
    )

The processed video is written to **resultVids/**, with the closest detected gesture displayed on the output frames.

The <u>TEST & DISPLAY RESULTS</u> section of the notebook contains examples using the included test videos.

**Limitations**
-

This project is an experimental prototype and currently has several limitations:

    + Recognition is limited to B, E, K, and W.
    + Only one hand is processed at a time.
    + Recognition depends on a color-coded glove or equivalent colored markers.
    + Calibration should be performed for the current lighting environment.
    + Background colors similar to the glove markers can interfere with segmentation.
    + The system relies on template matching rather than a learned gesture-recognition model.
    + The current implementation is designed primarily for front-facing hand gestures.

A clear, neutral background and consistent lighting provide the most reliable results.



*References*
-

* Video Reading:
    - https://opencv.org/reading-and-writing-videos-using-opencv/
    - https://www.geeksforgeeks.org/python/python-opencv-capture-video-from-camera/
    - https://painlessprogramming.com/work-with-camera-feed-in-python/
    - Mouse click interaction: https://www.geeksforgeeks.org/python/click-response-on-video-output-using-events-in-opencv-python/
    - Key press: https://www.geeksforgeeks.org/python/python-opencv-waitkey-function/
    - HSV color: https://medium.com/@dijdomv01/a-beginners-guide-to-understand-the-color-models-rgb-and-hsv-244226e4b3e3
    - Custom classes (to represent hand parts and hand): https://www.geeksforgeeks.org/python/python-classes-and-objects/
    - Color segmentation: 
        * https://realpython.com/python-opencv-color-spaces/
        * https://www.geeksforgeeks.org/computer-vision/choosing-the-correct-upper-and-lower-hsv-boundaries-for-color-detection-with-cv-inrange-opencv/
    - Contours: 
        * https://docs.opencv.org/3.4.20/d4/d73/tutorial_py_contours_begin.html
        * https://docs.opencv.org/3.4.20/d3/dc0/group__imgproc__shape.html#ga17ed9f5d79ae97bd4c7cf18403e1689a
        * https://stackoverflow.com/questions/44588279/find-and-draw-the-largest-contour-in-opencv-on-a-specific-color-python
    - Expanding contours (dilate)
        * https://stackoverflow.com/questions/55374305/how-to-get-an-expanded-or-contracted-contour-in-opencv
        * https://stackoverflow.com/questions/57157459/expanding-a-contour-in-python-cv2
    - Filling in gaps within contours
        * https://stackoverflow.com/questions/75925076/how-to-fill-small-gaps-in-a-contour-in-opencv
        * https://www.pythontutorials.net/blog/filling-contours-with-opencv-python/
        * https://docs.opencv.org/4.13.0/d9/d61/tutorial_py_morphological_ops.html
    - Euclidean Distance: 
        * https://pythonexamples.org/python-math-dist/
        * https://www.w3schools.com/python/ref_math_dist.asp
    - Masking & creating image
        * https://programming-review.com/python/opencv-create-image
        * 3 valued pix comparison: https://www.delftstack.com/howto/numpy/python-compare-arrays/
    - Image comparison
        * https://www.codegenes.net/blog/how-to-compare-2-images-in-python/
    - match/case 
        * https://www.w3schools.com/python/ref_keyword_case.asp
    - Terminal Argumants 
        * https://www.geeksforgeeks.org/python/run-function-from-the-command-line-in-python/
        * https://www.geeksforgeeks.org/python/command-line-arguments-in-python/
    - Dependency Installer
        * https://codelucky.com/install-packages-requirements-txt-pip/
