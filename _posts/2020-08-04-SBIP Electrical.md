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
<ol>

<p>
The total electrical diagram to address these design criteria is shown in Figure 1. In the rest of the post, the specific components from the circuit will be introduced.
</p>

<figure>
  <img src="/images/sbip_electrical/wiring_schematic.JPG" class="centered">
  <figcaption> Figure 1 - Wiring schematic for inverted pendulum robot.</figcaption>
</figure>
