---
layout: post
title: "Self-Balancing Inverted Pendulum - Electrical Design"
title_image: "/images/post_type_images/BJT.JPG"
date: 2020-08-09
---
<p>
In a <a href="https://malcolmhodgins.github.io/projects/2020/08/04/SBIP-Modelling">previous post</a>, I outlined the design requirements for the inverted pendulum (IP) problem and ran Matlab simulations to demonstrate how the IP controller can be tuned to create a stable closed-loop system. In this post, I will be translating the design objectives into system function objections as they pertain to the electrical system of the IP robot. The electrical system must accomplish the following tasks:
</p>

<ol>
  <li>Process data and execute instructions</li>
  <li>Measure pendulum angle and angular velocity</li>
  <li>Provide control action</li>
  <li>Measure cart position and cart velocity</li>
</ol>

<p>
The total electrical diagram to address these design criteria is shown in Figure 1. In the rest of the post, the specific components from the circuit will be introduced, the functions the component provides within the circuit, and the design choices that led to that specific component being chosen.
</p>

<figure>
  <img src="/images/sbip_electrical/wiring_schematic.JPG" class="centered">
  <figcaption> Figure 1 - Wiring schematic for inverted pendulum robot.</figcaption>
</figure>

<h2> MPU-6050 </h2>
<p>

</p>



<h2> Arduino Mega 2560 <h2>
<p>
As can be seen from Figure 1, all of the components in the IP robot are connected to the Arduino. Within this circuit, the Arduino serves to execute instructions, predetermined by the loaded code, based on the data received from the external components. On that basis alone, a vast number of micro-controllers or simply small computers could fit the criteria and be a candidate as the final component for this function of the IP robot. For example, the Arduino Uno seems to fit this criteria. Same with a Raspberry Pi. There are several more specific criteria that narrow down the available candidates significantly.
</p>
