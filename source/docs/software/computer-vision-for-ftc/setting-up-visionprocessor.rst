Setting up a VisionProcessor
============================

As of 2023 Centerstage, the official FTC Robot Controller project bundles OpenCV, and gives you a basic framework for this via `VisionPortal. <https://ftc-docs.firstinspires.org/en/latest/apriltag/vision_portal/visionportal_overview/visionportal-overview.html>`_ 
However, in order to write custom vision code for OpenCV we need to make our own VisionProcessor. 

Writing the code
----------------

In your IDE, make a new Java class and have it ``implements VisionProcessor``. 

Let's start by writing a very barebones ``VisionProcessor`` in a file named "SampleProcessor.java": ::

    // SampleProcessor.java
    package org.firstinspires.ftc.teamcode;

    import android.graphics.Canvas;

    import org.firstinspires.ftc.robotcore.internal.camera.calibration.CameraCalibration;
    import org.firstinspires.ftc.vision.VisionProcessor;
    import org.opencv.core.Mat;
    import org.opencv.core.Point;
    import org.opencv.core.Scalar;
    import org.opencv.imgproc.Imgproc;

    public class SampleProcessor implements VisionProcessor {
        /**
        * This gets called when a {@link org.firstinspires.ftc.vision.VisionPortal} first starts
        * receiving data for this VisionProcessor to process.
        *
        * <p>
        * The VisionProcessor can use this to set basic parameters such as the expected screen
        * resolution and the camera calibration constants. As many OpenCV pipelines do
        * region-of-interest selection relative to screen size, it is good practice to save these
        * parameters to recalibrate screen-proportion-to-pixel conversion functions and prevent the
        * pipeline from overrunning the bounds of the {@link Mat}s.
        * </p>
        *
        * <p>
        * Additionally, information about camera intrinsics is also provided.
        * </p>
        *
        * @param width the width of the camera data's input resolution in pixels
        * @param height the height of the camera data's input resolution in pixels
        * @param calibration camera calibration data.
        */
        @Override
        public void init(int width, int height, CameraCalibration calibration) {
            // save width/height here and ready the pipeline for operation here
        }

        /**
        * This method gets called to process OpenCV frames and is where the magic happens.
        *
        * @param frame The OpenCV frame received from the camera. It is a 3-channel {@link Mat} in the
        *              RGB color-space, unlike desktop OpenCV code which is often ordered BGR.
        *              <p> Drawing on this frame (e.g. with Imgproc.rectangle) will affect what gets
        *              drawn to the Driver Station/Robot Controller's vision preview, allowing you to
        *              annotate your pipeline output. </p>
        *              <p>Keep in mind that if you have multiple VisionProcessors, editing the input
        *              frame will edit it for the next VisionProcessor to be run which may affect
        *              its ability to detect things. Using FTC Dashboard can avoid this problem.</p>
        *              <p> As is true of most 2D computer graphics the (0, 0) pixel origin is in the
        *              top left and positive X is to the right and positive Y is down. </p>
        * @param captureTimeNanos a monotonically (that is, never decreasing) timestamp of when the
        *                         current frame was received in nanoseconds, which can be used to
        *                         calculate time deltas between frames.
        * @return a user-specified object. This can be anything, and is later passed to
        *         {@link #onDrawFrame} as the userContext argument. For 99% of users however it is
        *         sufficient just to return null.
        */
        @Override
        public Object processFrame(Mat frame, long captureTimeNanos) {
            // Pipeline written here.
            // For now, let's write "Hello, world!" to the frame we received.
            Imgproc.putText(
                    frame, // draw on the input frame so it gets displayed to the screen
                    "Hello, world!", // the text to write
                    new Point(0, 50), // bottom left corner of our text
                    Imgproc.FONT_HERSHEY_SIMPLEX, // the font to draw with -- "normal-size sans-serif"
                    1.0, // the font scale factor
                    new Scalar(255, 0, 0), // let's have our text be #ff0000 (red)
                    3 // thickness of 3 pixels
            );
            return null; // we aren't using onDrawFrame so returning null here is fine
        }

        /**
        * This method is called whenever a frame produced by {@link #processFrame} is to be drawn to
        * the robot controller/driver station screen.
        *
        * <p>It gives you an Android canvas context allowing you to annotate your pipeline output with
        * Android drawing primitives. These operate at a higher resolution than the OpenCV output and
        * can be used to annotate detections in finer detail. The official AprilTag detector for
        * example uses this method for that reason.
        * </p>
        *
        * <p>However, Android drawing primitives are more complicated to use than OpenCV's so for most
        * teams this method is not particularly useful.
        * The 2027 control system refresh will not use Android so any Android-specific work here
        * will eventually need translating over to OpenCV primitives.
        * </p>
        *
        * @param canvas an {@link android.graphics.Canvas} context object.
        *               Note that you should not query the width/height of the canvas from this object
        *               but rather query use onscreenWidth and onscreenHeight
        * @param onscreenWidth the width of the DS/CH viewport in pixels
        * @param onscreenHeight the height of the DS/CH viewport in pixels
        * @param scaleBmpPxToCanvasPx a scale factor between the size of the {@link Mat} and the pixel
        *                             scale used for drawing with the canvas. Basically,
        *                             canvasCoordinates = matCoordinates * scaleBmpPxToCanvasPx
        * @param scaleCanvasDensity a scaling factor to adjust e.g. text size. Relative to Nexus5 DPI.
        * @param userContext This is whatever got returned by the {@link #processFrame} call that this
        *                    method call is operating on. You can use this to provide frame-specific
        *                    information e.g. the timestamp of the frame or where things got detected,
        *                    but in practice just pulling the variables that the VisionProcessor is
        *                    currently holding is good enough for this.
        */
        @Override
        public void onDrawFrame(
                Canvas canvas,
                int onscreenWidth,
                int onscreenHeight,
                float scaleBmpPxToCanvasPx,
                float scaleCanvasDensity,
                Object userContext
        ) {
            // for 99% of users it is sufficient just to leave this function blank
        }
    }

(You don't need all the doc comments but they're there for your reference.)

Then, let's make a ``SampleProcessorOpmode`` to run the pipeline. :: 

    // SampleProcessorOpmode.java
    package org.firstinspires.ftc.teamcode;

    import android.util.Size;

    import com.qualcomm.robotcore.eventloop.opmode.OpMode;
    import com.qualcomm.robotcore.eventloop.opmode.TeleOp;

    import org.firstinspires.ftc.robotcore.external.hardware.camera.WebcamName;
    import org.firstinspires.ftc.vision.VisionPortal;

    @TeleOp(name = "Sample Processor Opmode")
    public class SampleProcessorOpmode extends OpMode {
        VisionPortal myVisionPortal;
        SampleProcessor processor;
        @Override
        public void init() {
            // Instantiate the VisionProcessor we just wrote.
            processor = new SampleProcessor();

            VisionPortal.Builder myVisionPortalBuilder;

            // Create a new VisionPortal Builder object.
            myVisionPortalBuilder = new VisionPortal.Builder();

            // Specify the camera to be used for this VisionPortal.
            // We're assuming the use of a USB webcam named "Webcam 1"
            // If you have a phone camera, one can use .setCamera(BuiltinCameraDirection.BACK) or
            // .setCamera(BuiltinCameraDirection.FRONT) instead.
            myVisionPortalBuilder.setCamera(hardwareMap.get(WebcamName.class, "Webcam 1"));

            // This is where we can add VisionProcessors.
            // If you want to also do AprilTags or TFOD you can add .addProcessor calls here.
            // The order of .addProcessor calls here determines the order in which each processor runs as well.
            myVisionPortalBuilder.addProcessor(processor);  // An added Processor is enabled by default.

            // Optional: set other custom features of the VisionPortal (4 are shown here).
            myVisionPortalBuilder.setCameraResolution(new Size(640, 480));  // Each resolution, for each camera model, needs calibration values for good pose estimation.
            myVisionPortalBuilder.setStreamFormat(VisionPortal.StreamFormat.YUY2);  // MJPEG format uses less bandwidth than the default YUY2 but has less fine detail
            myVisionPortalBuilder.enableLiveView(true); // Enable LiveView (RC preview).
            myVisionPortalBuilder.setAutoStopLiveView(true); // Automatically stop LiveView (RC preview) when all vision processors are disabled.

            // Create a VisionPortal by calling build()
            myVisionPortal = myVisionPortalBuilder.build();
        }

        @Override
        public void loop() {
            // At this point, we don't need to do anything here
            // The VisionPortal will automatically run while the opmode is active.
        }

        @Override
        public void stop() {
            myVisionPortal.close();
        }

    }

Testing the code
----------------

In order for the live view to work, your robot controller needs a video display. For a phone that's just the RC phone screen, but for a control hub you need to **plug in an HDMI monitor** to see the output. 

Once plugged in, start the opmode. You should see "Hello, world!" in red in the top left of the camera view. 

.. TODO: add an image to show what i mean

We've now made an opmode that uses OpenCV...albeit not to detect anything. 
And there's other problems with this setup that will be addressed in the following article to setup FTC Dashboard.
