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
  <code class="codebox">

  </code>
</section>
