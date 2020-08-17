---
layout: post
title: "Self-Balancing Inverted Pendulum - Mechanical Design"
title_image: "/images/post_type_images/robotics.JPG"
date: 2020-08-17
---
<section>
  <h2> Introduction </h2>
    <p>
      In the third installment of my Self-Balancing Inverted Pendulum series, I will be presenting the mechanical design portion of this project. The design objectives will be discussed as well as the fabrication process and limitations. This will be followed up by pictures of the mechanical drawings and assembly produced in Autodesk Inventor and presentation of the measurements that will be used in the Matlab model, from the <a href="https://malcolmhodgins.github.io/projects/2020/08/04/SBIP-Modelling" class="button_EIT">first post</a> of this series, to obtain the tailored full state feedback controller for this system. Without further adieu, lets jump in!
    </p>
</section>

<span><br></span>

<section>
    <h2> Design Objectives and Fabrication Process </h2>
      <p>
        It may be somewhat strange to group these two topics together, however, for this project, and my limited home fabrication capabilities, it was necessary. At my lab at home, I have a 3D printer so naturally I wanted to design a body for the inverted pendulum (IP) that could be 3D printed rather than try to fabricate some complex assembly from sheet metal or something to that effect.
      </p>
      <p>
        A benefit of the 3D printer too is that it can print a complex structure accurately all in one shot assuming the CAD model is set up correctly. There are some limitations, however. The size of the bed the printer deposits material has a finite size of 210 mm x 210 mm and in practice you usually stay a couple cm away from the bed edges. Our functional bed area is more like 180 mm x 180 mm. Additionally, due to the fact that the printer deposits material layer-by-layer by increasing the printer head height after each layer is completed, structure overhangs greater than 75 degrees require support material to be printed in order to support the structure. The added support material increases post-processing time for the part and can ruin the finish of a print. Ideally, we want the amount of support material to be at a minimum.
      </p>
      <p>
        Considering these limitations, we can begin defining the design objectives for the IP structure:
        <ul>
          <li>Securely house all system components i.e. Arduino Mega, MPU-6050, motors, etc.</li>
          <li>Minimize support material required</li>
          <li>Maximize center of mass height</li>
        </ul>
        The last objective comes out of the dynamics of the system. If the center of mass is higher, then the cart will not need to apply as large of a torque to provide correcting action to the system. Therefore, it should be easier to control the system without requiring more powerful motors.
      </p>
      <p>
        The last thing to touch on is the size of the pendulum. The modelling equations tell us a larger pendulum is easier to control, however, for a given angle of pendulum deviation the motors will also need to be faster in order to cover enough distance to correct the pendulum disturbance. Smaller pendulums can then handle larger deviations of the pendulum with the same motors which also happens to be more entertaining. Therefore, we have bounded the pendulum size. The pendulum needs to be just large enough to house all the system components and no more.
      </p>
</section>

<span><br></span>

<section>
  <h2> Mechanical Drawings </h2>
    <p>
      In the following section, drawings of the IP structure will be presented along with any relevant discussion. The IP structure was designed to fulfill the listed objectives from the previous section. Note that while the Matlab model assumes that the pendulum and cart are separate structures coupled through a pivot point, the pendulum-cart system in this project are rigidly connected. The pivot point of the pendulum and the axle of the cart are coincident.
    </p>
    <p>
      An overview of the pendulum housing is provided in Figure 1. There are three standoffs (or levels) on the structure. The topmost standoff, beneath what could be called the roof, holds the L298N motor diver and the 9 V battery. The second highest standoff holds the Ardunio Mega 2560 and includes cutouts on the wall so that the USB and power supply ports can be accessed while the IP is assembled. The third standoff is an open space to allow pins from the MPU-6050 to hang down. The MPU-6050 is bolted to some perf-board and then the perf-board is mounted to the holes seen on the standoff. At the bottom of the pendulum is the connection point to the cart. The teeth on the connection limit movement in the x and y directions and the holes on either side allow a bolt to threaded through to limit z movement.
    </p>
    <figure>
      <img src="/images/sbip_mechanical/drawing_1.PNG" class="centered">
      <figcaption>Figure 1 - Multiview of pendulum structure.</figcaption>
    </figure>
    <p>
      External dimensions for the pendulum structure are shown in Figure 2 as well as the height of the standoffs. Figure 3 goes into more detail of the standoffs by dimensioning the connection points to secure each of the system components to the pendulum structure. The dimensions for the connection points were determined using the datasheet for the component, when available, or by direct measurement, when unavailable.
    </p>
    <figure>
      <img src="/images/sbip_mechanical/drawing_2.PNG" class="centered">
      <figcaption>Figure 2 - External dimensions of pendulum structure.</figcaption>
    </figure>
    <figure>
      <img src="/images/sbip_mechanical/drawing_3.PNG" class="centered">
      <figcaption>Figure 3 - Sectional dimensioning of the pendulum standoffs.</figcaption>
    </figure>
    <p>
      Now we can move onto describing the structure for the cart. The cart will securely holf the gearmotors and wheels will be attached to the gearmotor output shaft to provide locomotive action for the IP control system. Figure 4 shows the dimensioning of the cart. At the top of the cart is inverse of the connection seen in Figure 1 which allows the cart and pendulum structures to be coupled together. The arch cutaways we sized to fit the diameter of the gearmotor body as well as some additional height added by the mounting brackets bought from <a href="https://www.pololu.com/product/2676" class="button_EIT">Pololu</a>. The mounting holes on the base of the cart were positioned so that the gearmotors mounted on the mounting brackets could be fit snuggly in the cart structure.
    </p>
    <figure>
      <img src="/images/sbip_mechanical/drawing_4.PNG" class="centered">
      <figcaption>Figure 4 - Multiview of the cart.</figcaption>
    </figure>
    <p>
      The diagram for the IP wheels is shown in Figure 5. The larger center hole is there to provide space for the gearmotor axle to protrude through and the four smaller holes are there to facilitate mounting the wheel to the mounting hub for the gearmotor axle. A shallow channel is present along the circumference of the wheels to improve the coefficient of friction of the wheels. 3D-printed plastic on a wood surface has very poor traction so rubber bands were fitted into the channel so that the wheels would not slip or spin when the motors begin rotating.
    </p>
    <figure>
      <img src="/images/sbip_mechanical/drawing_5.PNG" class="centered">
      <figcaption>Figure 5 - Multiview of the wheels.</figcaption>
    </figure>
    <p>
      Finally, Figure 6 shows the drawing of the IP assembly including models of the Arduino Mega 2560, L298N, 9V battery, gearmotors, mounting brackets, and mounting hubs for the wheels. The wired connections are left out of the diagram as well as the MPU-6050.
    </p>
