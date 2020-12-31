---
layout: post
title: "Measurement of Inverted Pendulum State Variables with OpenCV"
title_image: "/images/post_type_images/code.JPG"
date: 2020-12-30
---
<section>
  <h2> Introduction </h2>
    <p>
      I have long wanted to do a project using some configuration of computer vision. This fall term during one of my lab courses, I was introduced to OpenCV during one of our class discussions and soon afterwards, an idea popped into my head where I could combine the inverted pendulum I built last summer while learning about computer vision. If you're not familiar with OpenCV, I'll give a brief introduction. OpenCV is an open-source computer vision library with a vast number of machine learning algorithms tailored to visual sensing and processing applications. At a higher level, it allows your computer to autonomously perform vision tasks such as, for example, detecting where people or dogs are in an image, whether a person is wearing a mask or not, tracking moving objects, etc. The applications are endless. They really are.
    </p>
    <p>
      I saw an opportunity to learn a new skill, by exploring OpenCV, while also doing what any respectable engineer/researcher needs to do when they design and build a new system: quantify the performance of the system. Basically, last summer, I had designed a state-space controller for my robot and implemented it but I never got around to trying to measure exactly how well my system was performing. Especially as I tuned the controller for better and better results, its a non-trivial task determining whether one controller actually performs better than another. What I needed was a fairly precise and convenient way to measure the state variables of my system, such as pendulum angle, over time so that I could then perform some analysis on the results and make a numerically-informed decision about controller performance. What I set out to do with this project, besides learning how to use a computer vision-based application and sharpening my C++ skills, is use OpenCV to reliably measure the angle of my inverted pendulum robot over time and provide the data in a simple format for analysis.
    </p>
    <p>
      Honestly, this project went very smoothly which is probably a testament mostly to how easy OpenCV is to use. Their documentation is fantastic. Below I've embedded a YouTube video I put together to explain my project and the rest of this post will dive into the process, mechanics, and weak points of this project!
    </p>

    <iframe class="centered" width="560" height="315" src="https://www.youtube.com/embed/wsnC2qINjDE" frameborder="0" allow="accelerometer; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>
</section>

<span><br></span>

<section>
  <h2> Process Flow </h2>
    <p>
      As alluded to in my video, the flow in this project is broken up into three distinct sections. First, the video of the robot is recorded. Then, the video is analyzed using OpenCV where data about the angle of the pendulum is extracted, and the processed video files are saved. Following data extraction, python is used to create an animation of the plot to match the frames of the video in the original recording. Figure 1 shows this in graphical format.
    </p>
    <figure>
      <img src="/images/sbip_opencv/process_flow.PNG" class="centered">
      <figcaption class="centered">Figure 1 - Process flow.</figcaption>
    </figure>
</section>

<span><br></span>

<section>
  <h2> Inverted Pendulum Robot Preparation </h2>
    <p>
      As you probably noticed in the video, my robot has three red squares on the side. I added these squares to give my robot a distinguishable color on the body that can be used with OpenCV to extract information about the orientation of the pendulum i.e. its angle. The squares are not high-tech; in fact, I cut them out of the lid of a peanut butter jar I had around my house. If its simple and it works, then no reason to change it right?
    </p>
</section>

<span><br></span>

<section>
  <h2> OpenCV Script </h2>
  <p>
    Let's not beat around the bush; here's the script I used, but fair warning, its a little long and tailored to my specific application. After the script, I'll break it down so it's a little more digestible.
  </p>

<code>
#include <opencv2/opencv.hpp>
#include "opencv2/highgui/highgui.hpp"
#include "opencv2/imgproc/imgproc.hpp"
#include "opencv2/imgcodecs.hpp"
#define PI 3.14159265

using namespace cv;
using namespace std;

int main(int argc, char** argv)
{
    VideoCapture cap("C:/Users/malco/Desktop/OpenCV 4_5_1/ip_opencv/final_version/ip_test_video2.mp4"); //load video

    if (!cap.isOpened())  // if not success, exit program
    {
        cout << "Cannot open file" << endl;
        cin.get();
        return -1;
    }

    //initializing videowriter object
    int fps = 30;
    int frame_width = static_cast<int>(cap.get(CAP_PROP_FRAME_WIDTH));
    int frame_height = static_cast<int>(cap.get(CAP_PROP_FRAME_HEIGHT));
    Size frame_size(frame_width, frame_height);
    VideoWriter firstVideoWriter("C:/Users/malco/Desktop/OpenCV 4_5_1/ip_opencv/final_version/ip_test_video2_contours.avi", VideoWriter::fourcc('M', 'J', 'P', 'G'), fps, frame_size, true);
    if (firstVideoWriter.isOpened() == false) //check if the videowriter object was initialized
    {
        cout << "Cannot save the video to a file" << endl;
        cin.get(); //wait for any key press
        return -1;
    }

    VideoWriter secondVideoWriter("C:/Users/malco/Desktop/OpenCV 4_5_1/ip_opencv/final_version/ip_test_video2_thresholded.avi", VideoWriter::fourcc('M', 'J', 'P', 'G'), fps, frame_size, 0);
    if (secondVideoWriter.isOpened() == false) //check if the videowriter object was initialized
    {
        cout << "Cannot save the video to a file" << endl;
        cin.get(); //wait for any key press
        return -1;
    }

    //initializing data file
    ofstream myfile("C:/Users/malco/Desktop/OpenCV 4_5_1/ip_opencv/final_version/ip_test_video2_angle_data.txt");
    if (myfile.is_open() == false) { //checking if file opened
        cout << "Cannot open data file" << endl;
        cin.get();
        return -1;
    }

    //intializing values for control trackbars
    namedWindow("Control", WINDOW_AUTOSIZE); //create a window called "Control"
    int iLowH = 150;
    int iHighH = 179;

    int iLowS = 150;
    int iHighS = 255;

    int iLowV = 60;
    int iHighV = 255;

    int thresh = 150;

    //Create trackbars in "Control" window
    createTrackbar("LowH", "Control", &iLowH, 179); //Hue (0 - 179)
    createTrackbar("HighH", "Control", &iHighH, 179);

    createTrackbar("LowS", "Control", &iLowS, 255); //Saturation (0 - 255)
    createTrackbar("HighS", "Control", &iHighS, 255);

    createTrackbar("LowV", "Control", &iLowV, 255); //Value (0 - 255)
    createTrackbar("HighV", "Control", &iHighV, 255);

    const int max_thresh = 255;
    createTrackbar("Canny thresh:", "Control", &thresh, max_thresh);

    //Capture a temporary image from the camera
    Mat imgTmp;
    cap.read(imgTmp);

    //Create a black image with the size as the camera output
    Mat imgLines = Mat::zeros(imgTmp.size(), CV_8UC3);

    while (true)
    {
        Mat imgOriginal;

        bool bSuccess = cap.read(imgOriginal); // read a new frame from video
        if (!bSuccess) //if not success, break loop
        {
            cout << "Cannot read a frame from video stream" << endl;
            break;
        }

        Mat imgHSV;
        cvtColor(imgOriginal, imgHSV, COLOR_BGR2HSV); //Convert the captured frame from BGR to HSV

        Mat imgThresholded;
        inRange(imgHSV, Scalar(iLowH, iLowS, iLowV), Scalar(iHighH, iHighS, iHighV), imgThresholded); //Threshold the image

        //morphological opening (removes small objects from the foreground)
        erode(imgThresholded, imgThresholded, getStructuringElement(MORPH_ELLIPSE, Size(5, 5)));
        dilate(imgThresholded, imgThresholded, getStructuringElement(MORPH_ELLIPSE, Size(5, 5)));

        //morphological closing (removes small holes from the foreground)
        dilate(imgThresholded, imgThresholded, getStructuringElement(MORPH_ELLIPSE, Size(5, 5)));
        erode(imgThresholded, imgThresholded, getStructuringElement(MORPH_ELLIPSE, Size(5, 5)));

        secondVideoWriter.write(imgThresholded);//write thresholded frame to video file
        //imshow("Thresholded Image", imgThresholded); //show thresholded image

        //contouring process
        int thresh = 100;
        Mat canny_output;
        Canny(imgThresholded, canny_output, thresh, thresh * 2);
        vector<vector<Point> > contours;
        findContours(canny_output, contours, RETR_EXTERNAL, CHAIN_APPROX_SIMPLE); //identifies external contours in image
        vector<vector<Point> > contours_poly(contours.size());
        vector<Point2f>centers(contours.size());
        vector<float>radius(contours.size());
        for (size_t i = 0; i < contours.size(); i++)
        {
            approxPolyDP(contours[i], contours_poly[i], 3, true);
            minEnclosingCircle(contours_poly[i], centers[i], radius[i]);
        }
        Mat drawing = Mat::zeros(canny_output.size(), CV_8UC3);
        for (size_t i = 0; i < contours.size(); i++)
        {
            Scalar color = Scalar(0, 255, 0);
            drawContours(drawing, contours_poly, (int)i, color);
            circle(drawing, centers[i], (int)radius[i], color, 2); //drawing circles purely for viewing purposes
        }
        //end of contouring process

    //computing shape relative distances to each other in triangle
    //rounding floats to integers since values are >>100
    //maximum distance error is 0.5% per measurements which is insignificant
        if (contours.size() == 3) { //depending on how good the thresholding is, you may get more or less contours than expected. The following script relies on three contours
            int distances[3][3] = { {0,0,0},{0,0,0},{0,0,0} };
            for (size_t i = 0; i < 2; i++)
            {
                distances[i][0] = sqrt(pow(centers[i + 1].x - centers[i].x, 2) + pow(centers[i + 1].y - centers[i].y, 2));
                distances[i][1] = i + 1; //these two lines keep track of where the data came from in centers[i]
                distances[i][2] = i;
            }
            //looping to front of array
            distances[2][0] = sqrt(pow(centers[0].x - centers[2].x, 2) + pow(centers[0].y - centers[2].y, 2));
            distances[2][1] = 2; //these two lines keep track of indices used in centers[i]
            distances[2][2] = 0;

            int index1 = 0;
            int index2 = 0;
            //find median distance value on triangle
            if (((distances[0][0] > distances[1][0]) && (distances[0][0] < distances[2][0])) || ((distances[0][0] < distances[1][0]) && (distances[0][0] > distances[2][0]))) {
                index1 = distances[0][1];
                index2 = distances[0][2];
            }
            else if (((distances[1][0] > distances[0][0]) && (distances[1][0] < distances[2][0])) || ((distances[1][0] < distances[0][0]) && (distances[1][0] > distances[2][0]))) {
                index1 = distances[1][1];
                index2 = distances[1][2];
            }
            else if (((distances[2][0] > distances[0][0]) && (distances[2][0] < distances[1][0])) || ((distances[2][0] < distances[0][0]) && (distances[2][0] > distances[1][0]))) {
                index1 = distances[2][1];
                index2 = distances[2][2];
            }
            //compute angle between points wrt to vertical
            double param, ip_angle;
            param = (abs(centers[index1].x - centers[index2].x) / abs(centers[index1].y - centers[index2].y));
            if (centers[index2].x > centers[index1].x) {
                param = -1 * param;
            }
            ip_angle = atan(param) * 180 / PI; //pendulum angle in degrees

            Mat imgLines = Mat::zeros(canny_output.size(), CV_8UC3);
            line(imgLines, centers[index1], centers[index2], Scalar(0, 0, 255), 2); //drawing line along pendulum axis
            drawing = drawing + imgLines;

            myfile << ip_angle << "\n"; //writing angle to data file
            firstVideoWriter.write(drawing); //write contours and lines to video file
        }
        else {
            firstVideoWriter.write(drawing);//write contours and lines to video file
            double ip_angle = 180; //writes nonsensical value to data file
            myfile << ip_angle << "\n";
        }

        //uncomment these two lines if you want to see the frames being written the the video files
        //imshow("Contours", drawing); //show result of contouring and line drawing
        //imshow("Original", imgOriginal); //show the original image

        if (waitKey(1) == 27) //wait for 'esc' key press for 30ms. If 'esc' key is pressed, break loop
        {
            cout << "esc key is pressed by user" << endl;
            break;
        }
    }

    firstVideoWriter.release(); //close video file
    secondVideoWriter.release(); //close second video file
    myfile.close(); //close text file

    return 0;
}
</code>

  <p>
    At the start of the code, we're loading the libraries and video file we need as well as initializing the various files we need to write to. Below this part of the code, there is a bunch of lines related to trackbars. These trackbars play an integral part in isolating the red portion of the frames in my video so that the black white frames can be produced. When I first load the video, I adjust the <em>iLowH</em> and <em>iHighH</em> trackbars to center around the hue around corresponding to red. In my case I found 150 and 179 to be good values to use. A similar process is followed for the other trackbars. As these trackbars are adjusted, you're just trying to get as clean of an image as you can. That is, you want only the targets of interest, in my case, the red squares, to appear in the image. If background noise is getting through, then you need to try to tinker with the trackbars a little more or reconfigure your set up.
  </p>
  <p>
    <pre>
<code class="codebox">
#include <opencv2/opencv.hpp>
#include "opencv2/highgui/highgui.hpp"
#include "opencv2/imgproc/imgproc.hpp"
#include "opencv2/imgcodecs.hpp"
#define PI 3.14159265

using namespace cv;
using namespace std;

int main(int argc, char** argv)
{
    VideoCapture cap("C:/Users/malco/Desktop/OpenCV 4_5_1/ip_opencv/final_version/ip_test_video2.mp4"); //load video

    if (!cap.isOpened())  // if not success, exit program
    {
        cout << "Cannot open file" << endl;
        cin.get();
        return -1;
    }

    //initializing videowriter object
    int fps = 30;
    int frame_width = static_cast<int>(cap.get(CAP_PROP_FRAME_WIDTH));
    int frame_height = static_cast<int>(cap.get(CAP_PROP_FRAME_HEIGHT));
    Size frame_size(frame_width, frame_height);
    VideoWriter firstVideoWriter("C:/Users/malco/Desktop/OpenCV 4_5_1/ip_opencv/final_version/ip_test_video2_contours.avi", VideoWriter::fourcc('M', 'J', 'P', 'G'), fps, frame_size, true);
    if (firstVideoWriter.isOpened() == false) //check if the videowriter object was initialized
    {
        cout << "Cannot save the video to a file" << endl;
        cin.get(); //wait for any key press
        return -1;
    }

    VideoWriter secondVideoWriter("C:/Users/malco/Desktop/OpenCV 4_5_1/ip_opencv/final_version/ip_test_video2_thresholded.avi", VideoWriter::fourcc('M', 'J', 'P', 'G'), fps, frame_size, 0);
    if (secondVideoWriter.isOpened() == false) //check if the videowriter object was initialized
    {
        cout << "Cannot save the video to a file" << endl;
        cin.get(); //wait for any key press
        return -1;
    }

    //initializing data file
    ofstream myfile("C:/Users/malco/Desktop/OpenCV 4_5_1/ip_opencv/final_version/ip_test_video2_angle_data.txt");
    if (myfile.is_open() == false) { //checking if file opened
        cout << "Cannot open data file" << endl;
        cin.get();
        return -1;
    }

    //intializing values for control trackbars
    namedWindow("Control", WINDOW_AUTOSIZE); //create a window called "Control"
    int iLowH = 150;
    int iHighH = 179;

    int iLowS = 150;
    int iHighS = 255;

    int iLowV = 60;
    int iHighV = 255;

    int thresh = 150;

    //Create trackbars in "Control" window
    createTrackbar("LowH", "Control", &iLowH, 179); //Hue (0 - 179)
    createTrackbar("HighH", "Control", &iHighH, 179);

    createTrackbar("LowS", "Control", &iLowS, 255); //Saturation (0 - 255)
    createTrackbar("HighS", "Control", &iHighS, 255);

    createTrackbar("LowV", "Control", &iLowV, 255); //Value (0 - 255)
    createTrackbar("HighV", "Control", &iHighV, 255);

    const int max_thresh = 255;
    createTrackbar("Canny thresh:", "Control", &thresh, max_thresh);

    //Capture a temporary image from the camera
    Mat imgTmp;
    cap.read(imgTmp);

    //Create a black image with the size as the camera output
    Mat imgLines = Mat::zeros(imgTmp.size(), CV_8UC3);
</code>
    </pre>
  </p>
</section>

<span><br></span>

<section>
  <p>
    Now we move into the part of the script where all the action is happening. This code is structured with a while loop so that the code while loop through every frame in the video and apply the data extraction procedure to each frame. The boolean variable <em>bSuccess</em> is used to see if the end of the video has been reached. If it has, meaning there is no other frame available to be analyzed, then the program breaks.
  </p>
  <p>
    Entering into the while loop, a Blue-Green-Red (BGR) frame from the original video is read into a matrix first. Then the frame is converted to Hue-Saturation-Value (HSV) form so that we can apply the trackbars we recreated prior. The reason we do this is because having the colour represented by a single value, Hue, makes it extremely easy to center our trackbars around it. We then threshold the image using the trackbars values. This creates a binary image where pixels within the range of the trackbars are depicted as white and pixels outside the range are depicted as black. The red squares have now been isolated in the image and background information has been discarded.
  </p>
  <p>
    <pre>
<code class="codebox">
while (true)
    {
        Mat imgOriginal;

        bool bSuccess = cap.read(imgOriginal); // read a new frame from video
        if (!bSuccess) //if not success, break loop
        {
            cout << "Cannot read a frame from video stream" << endl;
            break;
        }

        Mat imgHSV;
        cvtColor(imgOriginal, imgHSV, COLOR_BGR2HSV); //Convert the captured frame from BGR to HSV

        Mat imgThresholded;
        inRange(imgHSV, Scalar(iLowH, iLowS, iLowV), Scalar(iHighH, iHighS, iHighV), imgThresholded); //Threshold the image

        //morphological opening (removes small objects from the foreground)
        erode(imgThresholded, imgThresholded, getStructuringElement(MORPH_ELLIPSE, Size(5, 5)));
        dilate(imgThresholded, imgThresholded, getStructuringElement(MORPH_ELLIPSE, Size(5, 5)));

        //morphological closing (removes small holes from the foreground)
        dilate(imgThresholded, imgThresholded, getStructuringElement(MORPH_ELLIPSE, Size(5, 5)));
        erode(imgThresholded, imgThresholded, getStructuringElement(MORPH_ELLIPSE, Size(5, 5)));

        secondVideoWriter.write(imgThresholded);//write thresholded frame to video file
        //imshow("Thresholded Image", imgThresholded); //show thresholded image
</code>
    </pre>
  </p>
  <p>
    The images also need to be filtered to clean up the white objects in the thresholded image. The erode() and dilate() functions are used to do this in two separate processes known as morphological opening and closing. In the sequence, morphological opening removes smaller white objects (noise) from the image while having less of an effect on the larger white objects and morphological closing removes small black objects that may be embedded in the larger white objects. The result is the very clean objects shown in the thresholded video in my YouTube video. I have included Figure 2 below to demonstrate this. There are a couple specs of white in the image before filtering that are removed by morphological opening and closing. Additonally, the edges of the larger white objects are cleaned up as little as well.
  </p>
  <figure>
    <img src="/images/sbip_opencv/unfiltered_and_filtered.PNG" class="centered">
    <figcaption class="centered">Figure 1 - Left image is the image before filtering and the right image is the image after filtering.</figcaption>
  </figure>
  <p>
    The last step in this portion of the code is to write the frames to the video file initialized in the previous section. There is also an option to display the thresholded frames as the video frames are being processed.
  </p>
</section>

<span><br></span>

<section>
  <p>
    With the frame thresholded, the next thing we need to do is locate the white objects in the frame. The <em>findContours()</em> function is used to locate the outside edges of the white objects by setting the contour type to <em>RETR_EXTERNAL</em>. Then information about the center-point and radius of each contour is extracted and a polygon approximation is created as well as a circle that encloses the contour. The circle is not really necessary but its better to look at in images. Finally the enclosing circles and polygon contours are drawn to a frame.
  </p>
  <p>
    <pre>
<code class="codebox">
        //contouring process
        int thresh = 100;
        Mat canny_output;
        Canny(imgThresholded, canny_output, thresh, thresh * 2);
        vector<vector<Point> > contours;
        findContours(canny_output, contours, RETR_EXTERNAL, CHAIN_APPROX_SIMPLE); //identifies external contours in image
        vector<vector<Point> > contours_poly(contours.size());
        vector<Point2f>centers(contours.size());
        vector<float>radius(contours.size());
        for (size_t i = 0; i < contours.size(); i++)
        {
            approxPolyDP(contours[i], contours_poly[i], 3, true);
            minEnclosingCircle(contours_poly[i], centers[i], radius[i]);
        }
        Mat drawing = Mat::zeros(canny_output.size(), CV_8UC3);
        for (size_t i = 0; i < contours.size(); i++)
        {
            Scalar color = Scalar(0, 255, 0);
            drawContours(drawing, contours_poly, (int)i, color);
            circle(drawing, centers[i], (int)radius[i], color, 2); //drawing circles purely for viewing purposes
        }
        //end of contouring process
</code>
    </pre>
  </p>
</section>

<span><br></span>

<section>
  <p>
    The last section of this code covers the process by which the axis of the pendulum is identified. In all honesty, it is quite application specific and will not be transferrable to most other projects so I will only describe the process at a high level. First the positions of the contours in the frame are extracted and the distances between each object are computed. Then, the computed distances are compared to locate the median value (which corresponds to the axis of the pendulum) and the contours corresponding to that distance are identified. Next, the angle between the two contour center-points compared to the vertical is computed and converted to degrees. We now have the angle of our pendulum in this frame! Then, a line is drawn between the two contour center-points and added to the frame with circles. Again, this is merely for viewing purposes. The last step is writing the frame to the video file (different from the thresholded video file!) and writing the pendulum angle data to a text file.
  </p>

  <p>
    <pre>
<code class="codebox">
//computing shape relative distances to each other in triangle
//rounding floats to integers since values are >>100
//maximum distance error is 0.5% per measurements which is insignificant
    if (contours.size() == 3) { //depending on how good the thresholding is, you may get more or less contours than expected. The following script relies on three contours
        int distances[3][3] = { {0,0,0},{0,0,0},{0,0,0} };
        for (size_t i = 0; i < 2; i++)
        {
            distances[i][0] = sqrt(pow(centers[i + 1].x - centers[i].x, 2) + pow(centers[i + 1].y - centers[i].y, 2));
            distances[i][1] = i + 1; //these two lines keep track of where the data came from in centers[i]
            distances[i][2] = i;
        }
        //looping to front of array
        distances[2][0] = sqrt(pow(centers[0].x - centers[2].x, 2) + pow(centers[0].y - centers[2].y, 2));
        distances[2][1] = 2; //these two lines keep track of indices used in centers[i]
        distances[2][2] = 0;

        int index1 = 0;
        int index2 = 0;
        //find median distance value on triangle
        if (((distances[0][0] > distances[1][0]) && (distances[0][0] < distances[2][0])) || ((distances[0][0] < distances[1][0]) && (distances[0][0] > distances[2][0]))) {
            index1 = distances[0][1];
            index2 = distances[0][2];
        }
        else if (((distances[1][0] > distances[0][0]) && (distances[1][0] < distances[2][0])) || ((distances[1][0] < distances[0][0]) && (distances[1][0] > distances[2][0]))) {
            index1 = distances[1][1];
            index2 = distances[1][2];
        }
        else if (((distances[2][0] > distances[0][0]) && (distances[2][0] < distances[1][0])) || ((distances[2][0] < distances[0][0]) && (distances[2][0] > distances[1][0]))) {
            index1 = distances[2][1];
            index2 = distances[2][2];
        }
        //compute angle between points wrt to vertical
        double param, ip_angle;
        param = (abs(centers[index1].x - centers[index2].x) / abs(centers[index1].y - centers[index2].y));
        if (centers[index2].x > centers[index1].x) { //incase the assignment of x-values in previous line was wrong
            param = -1 * param;
        }
        ip_angle = atan(param) * 180 / PI; //pendulum angle in degrees

        Mat imgLines = Mat::zeros(canny_output.size(), CV_8UC3);
        line(imgLines, centers[index1], centers[index2], Scalar(0, 0, 255), 2); //drawing line along pendulum axis
        drawing = drawing + imgLines;

        myfile << ip_angle << "\n"; //writing angle to data file
        firstVideoWriter.write(drawing); //write contours and lines to video file
    }
    else {
        firstVideoWriter.write(drawing);//write contours and lines to video file
        double ip_angle = 180; //writes nonsensical value to data file
        myfile << ip_angle << "\n";
    }

    //uncomment these two lines if you want to see the frames being written the the video files
    //imshow("Contours", drawing); //show result of contouring and line drawing
    //imshow("Original", imgOriginal); //show the original image

    if (waitKey(1) == 27) //wait for 'esc' key press for 30ms. If 'esc' key is pressed, break loop
    {
        cout << "esc key is pressed by user" << endl;
        break;
    }
}

firstVideoWriter.release(); //close video file
secondVideoWriter.release(); //close second video file
myfile.close(); //close text file

return 0;
}
</code>
    </pre>
  </p>
</section>

<span><br></span>

<section>
  <h2> Animating Plots in Python </h2>
    <p>
      Since I had these cool videos of the inverted pendulum being processed with OpenCV, I though it would be very cool to also have a plot that updated with the videos in synchronism. It turns out its very easy to do this in Python when making use of matplotlib and numpy. Here's the code and I have a brief explanation below.
    </p>
    <p>
      <pre>
<code class="codebox">
import matplotlib.pyplot as plt
import matplotlib.animation as animation
import numpy as np

y = np.loadtxt("C:/Users/malco/Desktop/OpenCV 4_5_1/ip_opencv/final_version/ip_test_video2_angle_data.txt")
fps = 30.0

# need to adjust the zero point of the measurements
sum = 0
for j in range(0,250):
    sum = sum + y[j]
initial_avg_y = sum/250.0
y = [i - initial_avg_y for i in y]

# create x values
x = np.arange(0,len(y)/fps,1.0/fps)

Writer = animation.writers['ffmpeg']
writer = Writer(fps=30, metadata=dict(artist='Me'), bitrate=1800)

fig = plt.figure()
plt.xlabel("Time (s)")
plt.ylabel("Pendulum Angle (degrees)")
plt.title("Inverted Pendulum Angle From Vertical vs. Time")
plt.subplots_adjust(left=0.15, right=0.9, top=0.85, bottom=0.2)
ax = fig.add_subplot(111)
line, = ax.plot([],[], '-')
ax.set_xlim(np.min(x), np.max(x))
ax.set_ylim(np.min(y), np.max(y))

def animate(i,factor):
    line.set_xdata(x[:i])
    line.set_ydata(y[:i])
    return line

K = 0.75 # any factor
ani = animation.FuncAnimation(fig, animate, frames=len(x), fargs=(K,),
                              repeat=False)
ani.save("angle_data_animation.mp4", writer=writer)
plt.show()
</code>
      </pre>
    </p>
    <p>
      Since my textfile was a single column corresponding to the pendulum angle in each frame, it was very straightforward to read in with numpy. No data formatting was required. The next thing that is done in the code is it corrects for any misalignment in the orientation of the camera compared to the pendulum, that is, for example, if the camera is slightly tilted to the right (In fact you can see this code in action in the video I posted to YouTube by looking at the original video of the pendulum. Yet the plot that is shown is centered around zero!). Next, a vector is created that is the same length as the amount of data available and each element corresponds to the period between frames for the given framerate of the video. In this case, my videos were recorded at 30 frames per second. Following this, the video writer and plot are created. The plot is animated using <em>animation.FuncAnimation()</em> and the animation is saved using <em>ani.save()</em>. That's it!. Pretty easy, right?
    </p>
</section>
