Setting up FTC Dashboard
========================

While you can use the Live View via to visualize your OpenCV pipelines through an HDMI connection or the driver station, there's some issues with the approach. 
Namely:

* since we call ``Imgproc.putText()`` on ``frame``, we're drawing on the same input frame that other ``VisionProcessor`` s would use which may impact their ability to detect things
* the way our pipeline image gets displayed is less than ideal -- our choices are between plugging in a monitor into the control hub, tapping the driver station screen to update the feed, or using a robot controller phone. All three of these options are not particularly ergonomic.
* we can't update parameters in real-time. What if we wanted to change "Hello, world!" to "Hello, FTC!" without recompiling the robot's code? That's valuable meeting time getting wasted away.

`FTC Dashboard <https://acmerobotics.github.io/ftc-dashboard>`_:highlight:`*` can address all of these issues. 
It allows users to setup live variable tuning in a web browser as well as do live camera streaming of our pipelines while letting us use OpenCV drawing primitives without clobbering the current frame for other VisionProcessors.

Installing FTC Dashboard - Android Studio
-----------------------------------------
See the official `Getting Started guide <https://acmerobotics.github.io/ftc-dashboard/gettingstarted>`_ for installation instructions.

The changes in your ``build.dependencies.gradle`` should look something like this: ::

    repositories {
        mavenCentral()
        google() // Needed for androidx
        maven { url = 'https://maven.brott.dev/' } // <-- this line is new
    }

    dependencies {
        implementation 'com.acmerobotics.dashboard:dashboard:0.4.16' // <-- this line is also new, replace the version number with whatever is most recent
        // .. leave the rest alone
    }

Compile and deploy the app. Connect to the robot controller network and verify http://192.168.43.1:8080/dash leads to a dashboard.

Installing FTC Dashboard - OnBot
--------------------------------

This is a bit of an unsolved problem. It is theoretically possible though.

The current issue is that FTC Dashboard relies on some mechanisms to start that only work if it is compiled into the app. However, doing so makes it impossible to compile onbot programs as trying to add the aar or jar file results in errors complaining that the classes already exist in the app.

it should be possible to integrate the class files into the global onbot jar file though. i don't know enough about gradle to achieve this goal at build time but my endgame is to try and compile an ftc-dashboard enabled app off of github actions so chromebook-encumbered teams can have a reasonable-quality dashboard.

there's no real good reason why ftc has been lacking an official dashboard. 
telemetry being able to display values is cool and all but there's a massive gap between teams that can fine-tune variables in real-time versus teams that don't. 
smartdashboard has been able to do this since before the android control system even existed, and frc programmers consider it vastly out of date. the mrc will probably fix this but in the meantime we gotta get something better.

also not entirely a fan of ftcdashboard's magic annotation processor approach but that's just me 


Writing a Dashboard-enabled VisionProcessor
-------------------------------------------

Lets revisiit...

::
    package org.firstinspires.ftc.teamcode;

    import android.graphics.Bitmap;
    import android.graphics.Canvas;

    import org.firstinspires.ftc.robotcore.external.function.Consumer;
    import org.firstinspires.ftc.robotcore.external.function.Continuation;
    import org.firstinspires.ftc.robotcore.external.stream.CameraStreamSource;
    import org.firstinspires.ftc.robotcore.internal.camera.calibration.CameraCalibration;
    import org.firstinspires.ftc.vision.VisionProcessor;
    import org.opencv.android.Utils;
    import org.opencv.core.Mat;
    import org.opencv.core.Point;
    import org.opencv.core.Scalar;
    import org.opencv.imgproc.Imgproc;

    import java.util.concurrent.atomic.AtomicReference;

    public class SampleDashboardProcessor implements VisionProcessor, CameraStreamSource {
        // This holds the last processed frame in our pipeline that will get sent to the dashboard.
        // It's initially filled with an empty frame.
        private final AtomicReference<Bitmap> lastFrame =
                new AtomicReference<>(Bitmap.createBitmap(1, 1, Bitmap.Config.RGB_565));
        @Override
        public void init(int width, int height, CameraCalibration calibration) {
            // let's init the last frame with a blank bitmap of the correct resolution.
            lastFrame.set(Bitmap.createBitmap(width, height, Bitmap.Config.RGB_565));
        }

        @Override
        public Object processFrame(Mat frame, long captureTimeNanos) {
            // Instead of directly drawing on the input frame, let's make a copy first.
            Mat output = frame.clone();

            // Let's still write "Hello, world!" to the frame we received.
            Imgproc.putText(
                    output, // draw on the input frame so it gets displayed to the screen
                    "Hello, world!", // the text to write
                    new Point(0, 50), // bottom left corner of our text
                    Imgproc.FONT_HERSHEY_SIMPLEX, // the font to draw with -- "normal-size sans-serif"
                    1.0, // the font scale factor
                    new Scalar(255, 0, 0), // let's have our text be #ff0000 (red)
                    3 // thickness of 3 pixels
            );

            // Display the frame to the dashboard.
            // We can call displayFrame on any intermediate Mat to visualize them in our pipeline.
            displayFrame(output);

            // Since our output image data got written to the bitmap already, we should free the output
            // Mat to prevent memory leaking.
            output.release();

            return null;
        }
        
        void displayFrame(Mat frame) {
            // Here's where the magic happens.
            // We create a new bitmap to display to the dashboard, and use
            // matToBitmap to convert our output frame
            Bitmap b = Bitmap.createBitmap(frame.width(), frame.height(), Bitmap.Config.RGB_565);
            Utils.matToBitmap(frame, b);
            lastFrame.set(b);
        }

        @Override
        public void getFrameBitmap(Continuation<? extends Consumer<Bitmap>> continuation) {
            continuation.dispatch(bitmapConsumer -> bitmapConsumer.accept(lastFrame.get()));
        }

        @Override
        public void onDrawFrame(
                Canvas canvas,
                int onscreenWidth,
                int onscreenHeight,
                float scaleBmpPxToCanvasPx,
                float scaleCanvasDensity,
                Object userContext
        ) {
            // this is still blank...
        }
    }



.. TODO: more things. Also displayFrame will probably break on binary Mats. 

.. I want to ramble about how you can still use the Params class even without FTC Dashboard.

more things

::
    import com.acmerobotics.dashboard.config.Config;
    @Config
    public static class PipelineConfig {
        static int HsvValue = ...;
    }