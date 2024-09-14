Apriltag Relocalization
=======================
.. note: untested. probably wrong. need to figure out how to actually do this

aka "Figuring out where you are on the field from AprilTags"

Camera position to robot center position
----------------------------------------

Assuming the camera is mounted facing dead straight forward on the robot, you can take the ftcPose of an AprilTagDetection and offset the x/y position to discover the robot's displacement from the apriltag.

assuming that +y is robot forward an +x is robot right side, let's say your camera is displaced from the robot center by :math:`c_x` and :math:`c_y` units.

we can take the values from the apriltag detection relative to the tag and offset the x and y to get the x/y offsets 


::
    AprilTagDectection detection;
    double robotTagX = detection.ftcPose.x + cameraOffsetX; // cx: camera x offset
    double robotTagY = detection.ftcPose.x + cameraOffsetY; // cy: camera y offset
    double robotTagDistance = Math.sqrt(robotTagX*robotTagX + robotTagY*robotTagY);


ok now that we know the relative displacement of the robot center to the apriltag center, we gotta figure out the displacement relative to the field coordinates. this means we gotta shift the orientation using the rotation of the robot. 

with the rotation of the robot (that we'll get from the IMU), we do something....

::
    double imuYaw; // radians
    double robotInnerAngle = Math.atan2(robotTagY, robotTagX);
    double fieldTriangleAngle = imuYaw + robotInnerAngle;
    double fieldXDisplacement = robotTagDistance * Math.cos(fieldTriangleAngle);
    double fieldYDisplacement = robotTagDistance * Math.sin(fieldTriangleAngle);

    double robotX = detection.fieldPosition.get(0) - fieldXDisplacement;
    double robotY = detection.fieldPosition.get(1) - fieldYDisplacement;

.. i keep trying to re-derive the math and i don't know if it works out.
   there's also the whole issue of relying on imu data here. i know that regular detection has some pose ambiguity.
   aaaaaaaaaaaaaaaaa

