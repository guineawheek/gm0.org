Color Blob Detection
====================

Overview
--------

This is one of the most basic and common tools using OpenCV in FTC challenges. 

The basic idea is to filter out and select specific colors that a webcam sees, group each color blob into a contour, and do analysis on the contour to figure out the position and/or orientation of objects of interest e.g. colored game elements for a randomization task.

The theory
----------

Generally, color blob detection follows these steps:

#. Blur the image to remove noise and less important finer details. In signal processing, we call this a "low-pass filter." Depending on how your pipeline is setup, this step is often optional.
#. Convert the image to a more useful colorspace for hue detection and will be more invaraint to different lighting conditions (usually HSV)
#. Threshold the image based on certain HSV ranges to produce a binary image of just the desired color (e.g. only green elements in the image)
#. Running contours detection to find each of the blobs of the desired color
#. Analyzing the contours to determine a result 


When is it worth using?
-----------------------

Color blob detection is not terribly sophisticated. 
Unlike a machine learning approach (e.g. Tensorflow), it is:

* easy to tune and adjust
* predictable (deterministic) in operation
* does not require a large training set
* easily extensible
* performant at high framerates (often at least 2x TensorFlow) on the control system
* lets you detect individual objects in a way that color fill comparison can't

However, it does have drawbacks, as it:

* assumes that the elements you're trying to detect are easily and consistently distinct in color from the background.
  This makes it generally better for game elements and field structures more towards the center of the field or close to the mat, rather than facing outwards past the drivers and into the stands.
* will lump multiple elements of the same color into one entity
* will confuse small objects with color splotches in the background (e.g. the color of shoes some drivers may be wearing)
* may require manual adjustment of thresholds under different lighting conditions
* is more complex than color fill comparison methods

Depending on your needs, other classical CV approaches or TensorFlow may fit them better. However, in many scenarios, you can get a clear view of the element you're trying to detect (historically, a randomization task).

A worked example: Relic Recovery
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

We can consider the 2017-2018 game, Relic Recovery. The jewels (colored whiffle balls where you displaced one in auto corresponding to your alliance color for points) are an ideal vision target, as you can position your robot in a starting position such that it always sees exactly one of the jewel, and the jewels are distinct colors (blue or red.)

However, being distinctly colored isn't enough to qualify as a good vision target. Many teams during Relic Recovery also considered trying to detect the walls of the cryptobox, an alliance-colored goal that you could score the 6" cubes into for points. The cryptobox does not make a particularly good vision target, however, as it has a clear back-plate against a field wall and breaks up the colored columns with red gaffer's tape to represent each scoring level. In practice this means that any OpenCV thresholding of the cryptobox will have to try and detect somewhere between 0-12 separate segments of box while also trying to distinguish it from whatever may be going beyond the field walls. As a result, teams were generally unsuccessful at trying to use vision here and ultimately odometry-based approaches for scoring in auto won out. 

Furthermore, trying to detect glyphs in the central glyph pit during auto was also extremely challenging, simply becaues the glyphs were grey and brown and thus similar in color to the field. They lacked super distinct edges so detecting glyphs with color blob detection was basically impossible on them. 

Tradeoff summary
^^^^^^^^^^^^^^^^

Historically, most games have some task that involves detecting the position of some game element in autonomous and said game elements are bright primary colors that are easy to threshold. Relic Recovery was a bit of an outlier here.

That said, **do not underestimate a simple color sensor!** While they have much less effective range and update slower, they are much easier to integrate and wire mechanically and do not require much Control Hub processing power. Teams with less experience in software may feel more comfortable using color sensors for randomization even if vision seems like the "meta", and still find success with a color sensor on a stick. 

Extracting the color we care about
----------------------------------

Blurring an image
^^^^^^^^^^^^^^^^^

Blurring an imnage lets us remove background noise from the image. Typically we only care about the overall color of elements on the field rather than the fine details (e.g. the edges and corners). Blurri

::
    Mat displayMat = new Mat();
    if (Params.shouldBlur) {
        Imgproc.blur(frame, displayMat, new Size(Params.blurSize, Params.blurSize));
    }
    // displayFrame(displayMat)



Converting the color space
^^^^^^^^^^^^^^^^^^^^^^^^^^

.. convert the image to HSV

HSV represents colors in terms of their hue (color), saturation (how intense the color is), and brightness instead of red/green/blue. 
This is preferable for color detection because we usually mostly care about the hue of the element, somewhat about the saturation, and next to nothing about the brightness. 
Using hue as an axis we measure also lets us be more specific about which colors we care about and lets us filter for colors that are not cleanly on the RGB axes (e.g. yellow).

::
    Mat hsvFrame = new Mat();
    Imgproc.cvtColor(hsvFrame, displayMat, Imgproc.COLOR_RGB2HSV);
    displayMat.release();


Basic image thresholding
^^^^^^^^^^^^^^^^^^^^^^^^

Unless you're dealing with trying to threshold the color red or brown, it's typically sufficient to have something like this:
::
    Mat thresh = new Mat();
    Core.inRange(thresh, new Scalar(Params.lowerHue, Params.lowerSat, Params.lowerVal), new Scalar(Params.upperHue, Params.upperSat, Params.upperVal), hsvFrame);
    hsvFrame.release();
    // your binary Mat is now in thresh
    // displayFrame(thresh);

and then adjust your HSV bounds accordingly.

Thresholding the color red
^^^^^^^^^^^^^^^^^^^^^^^^^^

Red is a special case as its hue values are both at the upper and lower ranges of hue as hue exists on a color circle.  You'll typically need two thresholding operations and then need to bitwise OR them together to get the desired result:
::

    Mat thresh = new Mat();
    Mat threshRed2 = new Mat();
    // Threshold from 0..upperHue
    Core.inRange(thresh, new Scalar(0, Params.lowerSat, Params.lowerVal), new Scalar(Params.upperHue, Params.upperSat, Params.upperVal), hsvFrame);
    // Threshold from lowerHue..180
    Core.inRange(threshRed2, new Scalar(Params.lowerHue, Params.lowerSat, Params.lowerVal), new Scalar(180, Params.upperSat, Params.upperVal), hsvFrame);
    hsvFrame.release();

    // Join the two masks together with an bitwise OR operation
    Core.bitwise_or(thresh, threshRed2, thresh);
    threshRed2.release();
    // displayFrame(thresh);

Contours versus color fill, a comparison
----------------------------------------

Now that we've thresholded for the color we care about, there's two paths we can proceed from here. 

There's two broad approaches we can take: Either we take a selected number of regions of interest and compare how much of our selected color is in them or we do a more broad search for contours.

The former is generally better for fixed-position randomization tasks where we generally know what parts of the image we should compare. 
The latter is better for more general object detection where we don't know where the objects fall, e.g. detecting blocks or balls in a pit of game elements. 

Color fill comparison
^^^^^^^^^^^^^^^^^^^^^

::
    // select the three boxes we care about; our "region of interest"
    Mat leftPosition = thresh.submat(Params.leftY, Params.leftY + Params.leftHeight, Params.leftX, Params.leftX + Params.leftWidth);

    Mat centerPosition = thresh.submat(Params.centerY, Params.centerY + Params.centerHeight, Params.centerX, Params.centerX + Params.centerWidth);

    Mat rightPosition = thresh.submat(Params.rightY, Params.rightY + Params.rightHeight, Params.rightX, Params.rightX + Params.rightWidth);

    // TODO: draw ROI rects

    int leftCount = Core.sumElems(leftPosition);
    int centerCount = Core.sumElems(centerPosition);
    int rightCount = Core.sumElems(rightPosition);
    // we don't need to free the submats because they're all backed by thresh

    thresh.free();

    if (leftCount > centerCount) {
        if (leftCount > rightCount) {
            objectPosition = Position.LEFT;
        } else {
            positionIndex = Position.RIGHT;
        }
    } else {
        if (centerCount > rightCount) {
            positionIndex = Position.CENTER;
        } else {
            positionIndex = POSITION.RIGHT;
        }
    }



Contours
^^^^^^^^

.. notes:
    Imgproc.CAHIN_APPROX_NONE: don't use this (wastes memory)
    Imgproc.CAHIN_APPROX_SIMPLE: good enough for blocks
    Imgproc.CHAIN_APPROX_TC89_L1: May deliver better performance for balls
    Imgproc.CHAIN_APPROX_TC89_KCOS: Likely not worth performance hit
    Sources: https://cvexplained.wordpress.com/2020/06/03/finding-and-drawing-contours/
    https://docs.opencv.org/4.x/d3/dc0/group__imgproc__shape.html#ga4303f45752694956374734a03c54d5ff


    For retr list, see: https://docs.opencv.org/4.x/d9/d8b/tutorial_py_contours_hierarchy.html
    Imgproc.RETR_EXTERNAL: outer contours. Nothing inner.
    Imgproc.RETR_LIST: Fine for most use cases
    Imgproc.RETR_CCOMP: alskdjf

::

    Mat hierarchy = new Mat();
    contours.clear();
    // find contours
    // for info on what hierarchy actually does, see the Mat docs and 
    // https://docs.opencv.org/4.x/d9/d8b/tutorial_py_contours_hierarchy.html
    // But for most cases we don't really care about hierarchy
    Imgproc.findContours(thresh, contours, hierarchy, Imgproc.RETR_LIST, Imgproc.CHAIN_APPROX_SIMPLE);

    // filter contours
    // there's a number of ways we can do this. 

    filteredContours.clear();

    for (contour : contours) {
        if (Imgproc.contourArea(contour) < Params.minContourSize) continue;
        // https://docs.opencv.org/4.x/dd/d49/tutorial_py_contour_features.html

        // Some contour processing functions require a MatOfPoint2f intsead of a MatOfPoint.
        // This code performs the conversion.
        MatOfPoint2f contour2f = new MatOfPoint2f();
        contour.convertTo(contour2f, CvType.CV_32FC2);


        // for basic object detection you probably want boundingRect
        // Rect rect = Imgproc.boundingRect(contour);
        // 
        
        // for rect rotation there's minAreaRect
        // RotatedRect rect = Imgproc.minAreaRect(contour2f);

        // for balls you should consider minEnclosingCircle
        // Imgproc.minEnclosingCircle​(MatOfPoint2f points, Point center, float[] radius)

        // for information about the center of a contour, regardless of shape
        // Moments p = Imgproc.moments(contour);
        // double x = p.get_m10() / p.get_m00();
        // double y = p.get_m01() / p.get_m00();


    }


.. We should probably split up this article into color fill and blob detection.
   I also want to touch on the issue of actually getting data _back into_ the robot control loop, 
   because that is a concept people struggle with. 

   Params is a class that's meant to be nested in this visionprocessor. It'll be annotated with @Config so FTC Dashboard
   can twiddle with it.