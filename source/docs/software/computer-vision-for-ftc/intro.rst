Introduction
============

.. note:: This entire guide is a massive WIP. It is still in active development. There will be missing parts, poorly worded paragraphs, and other undercooked things. Using the gm0 repo for this is primarily for convenience. 

This is a guide on how to do OpenCV object detection and AprilTags with FTC's VisionPortal. 

This assumes the use of Java throughout the tutorial, although the high-level principles are the same across any programming language. Given the relative dearth of Java-specific documentation for OpenCV, it's more important to understand at a high level what you wish to accomplish with your tools, and then do research on how to achieve them.

We assume the use of a REV Control Hub with a Logitech C270 webcam, a common and inexpensive configuration.

We also assume the use of **Android Studio** in order to setup **FTC Dashboard**. 

Java OpenCV
-----------

OpenCV is typically used in one of two programming languages: Python and C++. 
However, in FTC, we use Java, which means while there is OpenCV support, documentation, and examples, it's typically not as strong as those for Python or C++. 

OpenCV's Java bindings are much closer to the C++ API than the Python API, which is much more "high-level". 
Function signatures between Java and C++ are generally pretty similar and operate in the same way, and the Javadocs for many functions are the C++ ones hastily copypasted into Java.

When trying to figure out how to use OpenCV in Java, the C++ documentation is often pretty helpful and C++ examples can be more easily translated to Java than Python ones. 

Here are some general tips on how to navigate C++ docs for use in Java:

* The base unit of image data in Java is a ``Mat`` while in C++ it can be a variety of other things. Things like ``Point`` s and ``Scalar`` s are still a thing in Java.
* Unlike C++, Java does not have default arguments. Instead, method overloads are provided that will substitute in these values.
* One of the most challenging parts of Java OpenCV is figuring out which static class some function may belong to (e.g. ``org.opencv.core.Core`` or ``org.opencv.imgproc.Imgproc``). The name of the class a function comes from typically corresponds with the name of the C++ header file. Functions that require an ``#include <opencv2/imgproc.hpp>`` to use in C++ are static methods of the ``Imgproc`` in Java.
* Don't forget to ``.release()`` your ``Mat`` s when you're done with them.