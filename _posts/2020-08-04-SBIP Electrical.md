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
  <li><a href="#mpu6050">Measure pendulum angle and angular velocity</a></li>
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

<h2><a id="mpu6050"> MPU-6050 </a></h2>
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

<h2> Brushed DC Gearmotors and DC Motor Control </h2>
<p>
The second criteria the electrical system must fulfill is the ability to provide sufficient control action to correct disturbances to the IP. In the case of an IP robot, a motor of some sort is the clear option but depending on the specific system, vastly different control mechanisms can be employed to provide control action. For example, SpaceX is renowned for its upright-landing booster rockets. That system is clearly an IP and the control action to keep the rocket upright is provided by changing the direction the exhaust leaves the rocket so as to counteract movement at the nose of the rocket. Other IP projects balance the pendulum using the printer-head track from an dilapidated printer. In short, there really is no right answer when it comes to selecting the component which provides the control action so long as it satisfies the design criteria laid out for the project. Returning to our IP robot, there is a plethora of types of motors available to choose from; however, brushed DC motors have the advantage of usually being cheaper and more simple to operate.
</p>

<p>
We can further refine the selection of brushed DC motors we have to choose from by considering how the motor will be operated to control the IP. As stated in the <a href="https://malcolmhodgins.github.io/projects/2020/08/04/SBIP-Modelling" class="button_EIT">Matlab simulation post</a>, the average cart velocity will be zero. We also know that if the IP is balancing well, the wheels should barely be turning and only turning fast to provide correcting action after the IP has been bumped or disturbed. Therefore, motors which can turn slowly well will perform the best. This naturally leads to the consideration of a brushed DC gearmotor as a suitable candidate for the IP robot.
</p>

<p>
The American manufacturer <a href="https://www.pololu.com/category/115/25d-metal-gearmotors">Pololu</a> offers a range of brushed DC gearmotors available at various gear ratios and input voltages as shown in Figure 3. For this project, the 6V low-power 47:1 gear-ratio gearmotors were chosen. This gear ratio would offer a balance of suitable performance at both slow and fast rotation speeds. A voltage level of 6V was chosen for reasons related to best practices for controlling motors at slow speeds. DC motors suffer from substantial friction between the carbon brushes carrying current and the rotor. As such when a voltage is applied to the motor terminals, the voltage must build up to such a level that the electromotive force can overcome the friction force from the brushes. Using a voltage of 12 V in this case helps the motor receive the voltage necessary to overcome the internal friction faster as well as jerks the rotor into motion. Combined, this helps the motor to turn at lower speeds than if 6V was used. The only thing to watch out for is that the duty cycle of the pwm signal sent to the motors is halved such that the maximum allowable duty cycle is 50%. This helps prevent shortening the life of the motor due to winding overheating.
</p>

<figure>
  <img src="/images/sbip_electrical/gearmotors.JPG">
  <figcaption> Figure 3 - Brushed DC gearmotor selection from <a href="https://www.pololu.com/category/115/25d-metal-gearmotors">Pololu.</a></figcaption>
</figure>

<p>

</p>



<h2> Arduino Mega 2560 </h2>
<p>
As can be seen from Figure 1, all of the components in the IP robot are connected to the Arduino. Within this circuit, the Arduino serves to execute instructions, predetermined by the loaded code, based on the data received from the external components. On that basis alone, a vast number of micro-controllers or simply small computers could fit the criteria and be a candidate as the final component for this function of the IP robot. For example, the Arduino Uno seems to fit this criteria. A Raspberry Pi as well. There are several more specific criteria that narrow down the available candidates significantly.
</p>
