**CSCI_4261 | Intro to Comp Vision**

**Sign-Match ASL**(primitive)
-

Kaan Atilay | B00898029


a) How to run
-

The filter uses only commen libraries, but if needed dpenendencies can be installed using

    pip install -r dependencyInstaller.txt


The filter takes in 2 arguments, video path & video number (to record different result videos). Example:  the following code in vscode terminal will run the filter using one of the uploaded input videos and produce a video named result0
    
    python sign_Match_ASL.py inputVids/input_videoDemo.mp4 0

Filter created the result video in the folder: resultVids

b) Limitations & Requirements
-

- System vocabulary is limited to ['b', 'e', 'k', 'w']
- The system is limited to single hand per input video
- Each listed hand part should have a distinct colored representation either using the provided glove or 6 differently colored tapes. Colors used in development is respectively:
    * finger_base - yellow
    * Thumb - pink
    * Index Finger - orange
    * Middle Finger - blue
    * Ring Finger - green
    * Pinky Finger - purple
    * Palm - black



- Color callibration process should be performed to ensure accurate result on each run. This helps the program to behave consitently with different lightings.

**NOTE:** Avoid similar color to glove color in the background. For best results try to use clear white background.

**- The temp_DevImages, gesture_templates, inputVids, resultVids folders(with the content) MUST exist in the same directory as .py file.**
- The provided gesture templates include already normalized and ready to use templates. 

    -  For best results it is best to normalize raw represntation before new callibration. If needed, to normalize any gesture template run the block of code titled "NORMALIZING THE  RAW GESTURES" in the notebook submitted after adjusting the desired character as gestureName. Also to normalize and test the filter with new templates the code block titled "Saving raw represtations of each gesture" in the notebook can be used after adjusting the gestureName and pressed key.

-To see different reults of different inputs the codeblock titled "TEST & DISPLAY RESULTS" in the notebook can also be used.

**NOTE:** Different and more detailed(frame number, score, character, corresponding frame, extended debug list if needed) results can be viewed using the code block under the title TEST & DISPLAY RESULTS


c) Project Explanation
-

<u>1- Callibration Description</u>

<u>Keys</u>

    finger base tape - [y]
    thumb - [a]
    index - [s]
    middle - [d]
    ring - [f]
    pinky - [g]
    palm - [h]

--> System starts with callibration process using the live webcam. To callibrate color for each part;

    + While holding the hand still press the corresponding key for the hand-part you want to callibrate, then left-click that hand part in the webcam view. The result can be viewed in terminal
        - Ex: Press 'a' key and then left-click thumb to callibrate/set  the color for thumb acroess the current run.

NOTE: The manual for callibration can be toglled on/off by pressing the 'm' key. 


**<u>Process</u>**
- Callibrate colors for all 6 hand_parts and finger_base tape.
- For each part in handDict detect tip & base/ center using calibrated HSV values
    * Fnd the contour with max area & fill --> part's region
    * Find finger base regions & dilate --> avg of intersaction = finger_base & furthest point in region = finger_tip 
- After all 6 parts found --> create & normalize detected hand using palm width and center
- Compare captured gesture with each gesture_template
    * Compute MSE as similarty score
    * Penalize simScore based on the count of mismatched visible hand_parts
- Return consistently matched characters that are below the similarty threshold. 
    * Output a video displaying the closest current match on top left.

**PseudoCode & Code Explanation**
-

<u>Global Variables</u>
* partList  -   List of hand-part names & callibration mode 
* calibMode     -   String value to store selected callibration mode (selected from partList)
* click_location    -   tuple to store coordinates(x,y) and HSV values of clicked pixel area (temproraluy stored after each click)
* hand_dict     -       Global dictionary list to store each created individual hand-parts
* complete_hand     -       Represents the hand instance after all hand_parts callibrated & detected
* baseTapeHSV_lowT, baseTapeHSV_highT       -       Represents the color threshold represnting each finger base
* TEMPLATE_LIST     -       Stores the character & template img tuples
* TEMPLATE_PARTS    -   Stores the visible parts of templates
* SIM_THRESHOLD 
* MIN_REQ_FRAMES 
**Classes & Functions**
-
**<u>"hand_part" class --></u>**  Represnets each individual hand part object/instance
    
<u>- Attributes</u>

    + name
    + self.hsv_low_threshold   |  lower HSV thresholds accepted for corresponding color  
    + self.hsv_high_threshold  |  upper HSV thresholds acceptedfor corresponding color  
    + color                    |  combinatio of both thresholds in dictionaty form
    + region                   |  Represents the binary region mask of the part
    + visibility               |  Boolean value to represent if finger detected on current + frame
    + baseP ('finger' part)    |  Coords of of corresponding points  
    + tipP  ('finger' part)
    + centerP ( 'palm' part)
    
    NOTE: Region, visibility, centerP/baseP/tipP is setusing fintTipandBase function

<u>- Functions  </u>

    + Constructor   |   hand_part( partName, colorValue )

    + findeTipandBase( curFrame, curFrame_hsv, baseTapeHSV_lowerT, baseTapeHSV_higherT )
    --> Detects the corresponding hand_part in the current frame

        1. Find the region of corresponding part based on its color thresholds
            + Also fills in the small holes within the detected region
        2. Determine the contour of the part based on its region(step 1)
            + Find the index of contour that covers max area
        3. Fill in the found contour  & assign as part's *region*
        - For fingers:
            4. Find the region of finger_base regions
            5. Dilate the region contour and finger_base outwards to intersect with each other
            6. Assign average of intersecting points as --> *baseP*
               Assign furthest point to baseP within the region as --> *tipP*
            7. If both baseP & tipP is found 
                - set hand_part as --> *visible*
                - Draw pointson each point & connect
        - For palm
            4. Determine mean/average of palm region --> *centerP*
            5. If centerP is found
                - set palm visibility as --> *visible*
                -Draw circle at centerP
    NOTE: Each created mask for each part is saved respectively under the folders temp_DevImages/masks/{partName}.
    Saved masks: 
        * region_mask
        * contour_mask
        * complete_region_mask (after filling in the holes within region)
        * dilated_contour_mask
        * finger_base_mask
        * dilated_finger_base_mask
        * intersection_mask
    Example: temp_DevImages/msaks/index folder includes the 7 masks created for index finger part
        

**<u>"hand" class --></u>**  Represnets the hand object/instance
    
<u>- Attributes</u>

    + thumb (type = hand_part)
    + indexFinger (type = hand_part)
    + middleFinger (type = hand_part)
    + ringFinger (type = hand_part)
    + pinkyFinger (type = hand_part)
    + palm (type = hand_part) 
    
<u>- Functions  </u>
    
    + Constructor | hand(thumb, index, middle, ring, pinky, palm)
        
    + iterateHand()
        --> Returns a lsit of each hand part (in order)

    + normalizeHand()
        --> Captures the gesture of detected complete hand and normalizes according palm width to store in a fixed size gesture_mask
        --> *Returns: * the normalized gesture_mask & lsit of transformed part region mask for visible parts  (used to compare when computing similarity)
            
            1. Create 400x400 gesture_mask & set ideal origin position & palm width
            2. Determines the location of current origin(palm center) & palm_width
            3. Determine the necessary transformation matrix
                + Differnce between target origin and current origin --> translation values
                + Difference between target palm width and curren palm width --> scaling ratio
                + Combine them in single transofrmation matrix 
                NOTE: System finds the scaled current origin value after scaling to translate the gesture appropriately
            4. Apply transformation to each visible hand part
            5. Draw the normalized hand parts ony by one in the gesture_mask with respective colors.

+ <u>Warping_affine (transofrmationatrix, curRegion, output_H, output_W)</u>

    --> This function applis the passed in transformationMatrix to the passed in region mask point by point. This helps to create and align the normalzied gesture templates.

+ <u> find_active_coords_rgb(mask): </u> 

    -->Takes in a 3 chanlled(BGR) image and teruns the cooridnates(row, col) of pixels that are not black

+ <u> find_gesture(curGestureMask, transformedHandPartRegionList) </u>

    --> This function takes in a normalized gesture mask  & part_region_mask --> returns a similarty score against every gesture in system vocabulary (TEMPLATE_LIST) 

        1. gestuerMaskCoords <-- colored pixels of passed in gesture mask
        2. For each gestue tempalte in sys vocabulary: curTemplateCoord <-- colored pixels of current template
        3. Combine all pixels to compare(result of step 1 & 2) --> totActiveCoords
        4.Retrieve all totActiveCoords pixels in passed gesture mask and tempalte in respective lists based  
        5.Compute mse(mean squared error) based on step-4
        6. Penalize mse (by 200 per mismatched part) if a hand_part is
            - visible in current gesture but shouldn't be compared to current template
            - should be visible according to curTemplate but it is NOT in current gesture

+ <u>"process_click" function --></u> Processes the left-mouse click 
    
        1- Captures and small pixel area around the clicked pixel 
        
        2- Creates the lower and higher color threshold by respectively +/- 5,20,40 (for finger base 5,60,70) to current colorValue(hue, saturation, brightness)
        
        3- Creates the corresponding hand part according to the color values computed on step-2 & current calibration state at the time user clicked and saves it in *handDict*





