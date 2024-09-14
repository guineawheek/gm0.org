Setting up a webcam
===================

This guide will explain how to setup a webcam for use with computer vision in FTC.

Selecting a webcam
------------------
Any webcam compatible with VisionPortal should work here. 
This tutorial is based on the Logitech C270 -- a $20 webcam purchasable from the official FTC storefront. 
There are better cameras that can be bought for more money. 
For more information on camera selection, see `the official FTC Docs page on this. <https://ftc-docs.firstinspires.org/en/latest/apriltag/vision_portal/visionportal_webcams/visionportal-webcams.html>`_


While you *can* use the cameras on a robot controller phone if you don't have a Control Hub, this forces you to mount your phone where you want your webcam which is not always desirable and can be annoying to use. Even if you are still using a robot controller phone we still recommend you use a webcam.

Mounting your webcam
--------------------
Webcams are designed for use on computer monitors and not robots and so do not typically have nice mounting patterns.
It's still important, however, that your webcam is mounted securely to your robot especially with AprilTags which are position sensitive. 
FTC teams have developed and published `3D-printed mounts for common webcams <https://www.yeggi.com/q/logitech+webcam+mount+ftc/>`_ on platforms like Thingiverse, although it may be a good exercise for the team to CAD and print one themselves.

That said, there's always zipties and tape (not recommended).

Wiring your webcam
------------------

Plug the webcam into the USB 3.0 port on a Control Hub, or into a USB hub connected to the rest of your control system. Direct connection will have better (albeit largely not-noticable) latency.

.. warning:: Avoid plugging in USB webcams into the USB 2.0 port on your Control Hub, as static electricity on the USB line can cause the internal Wi-Fi radio on the Control Hub to reset, **resulting in a disconnect!** The radio is internally also connected via USB 2.0 and the power/ground lines are not isolated from the external port.

If the webcam is mounted to the drivebase, it's typically sufficient to make sure the wire doesn't get crunched somewhere or tangled in a roller. 

.. todo: figure out how to word this not stupid

If mounted to an extension additional considerations are required -- while it's possible to make your own cables, you need to shield them (and not violate impedance) to work consistently so it's easier just to buy an existing USB extension cable. 
Just don't forget to make the connections secure.

That said, `USB 2.0 is quite resiliant from a signal integrity standpoint. <https://youtu.be/aAqJYWu5Y8c>`_

Setting up the robot configuration
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Once the webcam is plugged into your Control Hub, select **Configure Robot** either on the Control Hub or the driver station. 
Select **Edit** for the currently active configuration and select **Scan**. 

A USB webcam should be listed under **USB Devices in Configuration**. If one doesn't appear, check your wiring. 
By default, this webcam is assigned a name like **Webcam 1**, but you can rename it to whatever you wish. 
When done, press **Save.**

Like all USB devices in FTC, webcams are uniquely identified by their serial number so if you replace your webcam on your robot you will need to repeat this configuration process and rename the resulting webcam to match your code.
