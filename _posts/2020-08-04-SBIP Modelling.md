---
layout: post
title: "Self-Balancing Inverted Pendulum - State Space Modelling and Matlab Simulations"
date: 2020-08-04
---

<p>
With a surplus of available time due to COVID-19, I decided to put to use the electronics, programming, control theory skills to the test by creating a robot consisting of only two wheels that is able to stand straight-up on these wheels indefinitely (imagine trying to balance a pencil on its eraser with only a finger). This is a well-known control problem since it is inherently an unstable system to begin with and requires careful consideration of the controller design in order to close the loop, allowing the pendulum to balance. I set out on this project with the following objectives in mind:
</p>

<p>
  <ul>
    <li>
      <span> - Balance indefinitely i.e. be a stable closed-loop system </span>
    </li>
    <li>
      <span> - Able to resist and dissipate bumps to the pendulum </span>
    </li>
    <li>
      <span> - Hold a position on the surface i.e. average velocity of the robot is zero </span>
    </li>

  </ul>
</p>

<p>
My introduction to control theory over the course of my degree was limited to PID (Proportional-Integral-Derivative) control so I knew I would also need to do some research on alternative control methods if I was to fulfill the design objectives I previously laid out. For one, the number of input variables will be greater than one since I will need to control the orientation of the pendulum as well as the location of robot in space. Beyond nesting PID loops to control each variable, I didn't know which controller design technique would be the most helpful.
</p>

<p>
After spending sometime online researching controller design, I came across a <a href="http://ctms.engin.umich.edu/CTMS/">Matlab Controls Tutorial</a> website by Rick Hill of Michigan University which provided detailed explanations of modelling techniques and applications to a variety of mechanical control problems, including the inverted pendulum problem.
</p>
