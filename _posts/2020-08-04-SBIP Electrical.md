---
layout: post
title: "Self-Balancing Inverted Pendulum - Electrical Design"
title_image: "/images/post_type_images/BJT.JPG"
date: 2020-08-09
---
<p>
In a <a href="https://malcolmhodgins.github.io/projects/2020/08/04/SBIP-Modelling" class="button_EIT">previous post</a>, I outlined the design requirements for the inverted pendulum (IP) problem and ran Matlab simulations to demonstrate how the IP controller can be tuned to create a stable closed-loop system. In this post, I will be translating the design objectives into system function objections as they pertain to the electrical system of the IP robot. The electrical system must accomplish the following tasks:
</p>

<ol>
  <li>Process data and execute instructions</li>
  <li>Measure pendulum angle and angular velocity</li>
  <li>Provide control action</li>
  <li>Measure cart position and cart velocity</li>
</ol>

<p>
The total electrical diagram to address the design criteria is shown in Figure 1. In the rest of the post, the specific components from the circuit will be introduced, the functions the components provide within the circuit, and the design choices that led to that specific component being chosen, will be discussed.
</p>

<figure>
  <img src="/images/sbip_electrical/wiring_schematic.JPG" class="centered">
  <figcaption> Figure 1 - Wiring schematic for inverted pendulum robot.</figcaption>
</figure>

<h2> MPU-6050 </h2>
<p>
One of the fundamental measurements that must be performed in order to implement the full-state feedback controller described in the <a href="https://malcolmhodgins.github.io/projects/2020/08/04/SBIP-Modelling" class="button_EIT">Matlab simulation post</a> is the measurement of the angle of the pendulum. The component which fulfills task 1. of the electrical functions a.k.a. measures the angle of the pendulum, must be accurate, fast, and compact in order to be deployed on the IP robot. Luckily, there are a number of prefabricated modules cheaply available. The MPU-6050 is easily available at many online stores, such as <a href="https://www.amazon.ca/Gikfun-MPU-6050-Accelerometer-Gyroscope-EK1091x3C/dp/B07JPK26X2/ref=sr_1_3?dchild=1&keywords=mpu-6050&qid=1597340164&s=electronics&sr=1-3" class="button_EIT">Amazon</a>, is clearly compact, as shownm in Figure 2, and it is quite fast. The MPU-6050 houses a 3-dimensional gyroscope as well as a 3-dimensional accelerometer which is enough to construct accurate positioning information about the pendulum angle when a Kalman or complementary filter is used on the data. However, it is not necessary to even write code for this module to unpack the data since the MPU-6050 houses a digital-motion-processor (DMP) which outputs accurate position information directly. When the data is ready, the "INT" pin in Figure 2, changes and the micro-controller can then read the data out. Note that the DMP only outputs angle data and not angular velocity data. On the micro-controller, this means a first order backward differentiation must be performed on the measured angle data in order to obtain angular velocity data as well.
</p>

<figure>
  <img src="/images/sbip_electrical/mpu6050.JPG" class="centered">
  <figcaption> Figure 2 - MPU6050 module.</figcaption>
</figure>

<p>
In terms of the electrical requirements of the MPU-6050, the "Vcc" pin takes 3.3 V which can be provided by the micro-controller, the "SCL" and "SDA" pins must be connected to "SCL" and "SDA" pins on the micro-controller to establish I2C communication, and the "INT" pin should be connected to an interrupt pin.
</p>



<h2> Arduino Mega 2560 </h2>
<p>
As can be seen from Figure 1, all of the components in the IP robot are connected to the Arduino. Within this circuit, the Arduino serves to execute instructions, predetermined by the loaded code, based on the data received from the external components. On that basis alone, a vast number of micro-controllers or simply small computers could fit the criteria and be a candidate as the final component for this function of the IP robot. For example, the Arduino Uno seems to fit this criteria. A Raspberry Pi as well. There are several more specific criteria that narrow down the available candidates significantly.
</p>
