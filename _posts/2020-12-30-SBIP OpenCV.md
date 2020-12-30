---
layout: post
title: "Measurement of Inverted Pendulum State Variables with OpenCV"
title_image: "/images/post_type_images/code.JPG"
date: 2020-12-30
---
<section>
  <h2> Introduction </h2>
    <p>
      I have long wanted to do a project using some configuration of computer vision. This fall term during one of my lab courses I was introduced to OpenCV during one of our class discussions and soon afterwards an idea popped into my head where I could combine the inverted pendulum I built last summer while learning about computer vision. If you're not familiar with OpenCV, I'll give a brief introduction. OpenCV is an open-source computer vision library with a vast number of machine learning algorithms tailored to visual sensing and processing applications. At a higher level, it allows your computer to autonomously perform vision tasks such as, for example, detecting where people or dogs are in an image, whether a person is wearing a mask or not, tracking moving objects, etc. The applications are endless. They really are.
    </p>
    <p>
      I saw an opportunity to learn a new skill, by exploring OpenCV, while also doing what any respectable engineer/researcher needs to do when they design and build a new system: quantify the performance of the system. Basically, last summer, I had designed a state-space controller for my robot and implemented it but I never got around to trying to measure exactly how well my system was performing. Especially as I tuned the controller for better and better results, its a non-trivial task determining whether one controller actually performs better than another. What I needed was a fairly precise and convenient way to measure the state variables of my system, such as pendulum angle, over time so that I could then perform some analysis on the results and make a numerically-informed decision about controller performance. What I set out to do with this project, besides learning how to use a computer vision-based application and sharpening my C++ skills, is use OpenCV to reliably measure the angle of my inverted pendulum robot over time and provide the data in a simple format for analysis.
    </p>
    <p>
      Honestly, this project went very smoothly which is probably a testament mostly to how easy OpenCV is to use. Their documentation is fantastic. Below I've embedded a YouTube video I put together to explain my project and the rest of this post will dive into the process and mechanics of how this project works!
    </p>

    <iframe class="centered" width="560" height="315" src="https://www.youtube.com/embed/wsnC2qINjDE" frameborder="0" allow="accelerometer; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>
</section>

<span><br></span>

<section>
  <h2> Process Flow </h2>
    <p>
      As alluded to in my video, the flow in this project is broken up into three distinct sections. First, the video of the robot is recorded. Then, the video is analyzed using OpenCV, data about the angle of the pendulum is extracted, and the processed video files are saved. Following data extraction, python is used to create an animation of the plot to match the frames of the video in the original recording. Figure 1 shows this in graphical format.
    </p>
    <figure>
      <img src="/images/sbip_opencv/process_flow.PNG" class="centered">
      <figcaption class="centered">Figure 1 - Process flow.</figcaption>
    </figure>
</section>

<span><br></span>

<section>
  <h2> OpenCV Script </h2>
  <p>
    Let's not beat around the bush; here's the script, but fair warning, its a little long. After the script, I'll break it down so it's a little more digestible.
  </p>
  <p>
    <pre>
<code class="codebox" style="height:1000px;overflow-y:scroll;">
#include <iostream>
#include <string>
#include <opencv2/opencv.hpp>
#include "opencv2/highgui/highgui.hpp"
#include "opencv2/imgproc/imgproc.hpp"
#include "opencv2/imgcodecs.hpp"
#include <math.h>
#include <fstream>
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
        vector<Rect> boundRect(contours.size());
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
    </pre>
  </p>
  <p>
    At the start of the code, we're loading the libraries we need as well as initializing the various files we need to write to.
  </p>
  <p>
    <pre>
<code class="codebox">
#include <iostream>
#include <string>
#include <opencv2/opencv.hpp>
#include "opencv2/highgui/highgui.hpp"
#include "opencv2/imgproc/imgproc.hpp"
#include "opencv2/imgcodecs.hpp"
#include <math.h>
#include <fstream>
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
