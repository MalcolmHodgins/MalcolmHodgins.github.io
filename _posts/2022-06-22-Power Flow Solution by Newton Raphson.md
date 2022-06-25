---
layout: post
title: "Development of Newton Raphson Power Flow Solution Code"
title_image: "/images/post_type_images/code.JPG"
date: 2022-06-22
---
<section>
  <h2> Introduction </h2>
    <p>
      For any power system to operate under normal balanced three-phase steady state conditions, it must be ensured that the generation is sufficient for the load on the system, that bus voltages do not deviate significantly from rated values, generators operate within their real and reactive power limits, and transmission lines and transformers do not become overloaded. The basic tool to investigate satisfaction of these criteria is a power flow calculation for given power system parameters. In this post, we will be looking into power flow solutions utilizing the Newton Raphson method and briefly reviewing code I developed to implement the power flow solution method.
    </p>
</section>

<span><br></span>

<section>
  <h2> The Math </h2>
    <p>
      As we start, I should note I have drawn from the textbook Power System Analysis and Design by D.Glover, M. Sarma, T. Overbye to aid in the explanation of the calculation and for the formula images. There are a couple different pieces that are required in order to begin this calculation for example, a given bus will have a voltage (V), a voltage angle (delta), and a certain real power (P) and reactive power (Q) while operating in a power system. For the calculation, the variables are rolled into the vectors shown below.
    </p>
    <figure>
      <img src="/images/power_flow_solution_NR/V_delta_P_Q_vector.png" class="centered">
      <figcaption class="centered"> Figure 1 - Voltage, voltage angle, real power, and reactive power vectors.</figcaption>
    </figure>
    <p>
      Where V, P, and Q are in per-units but the voltage angle, delta, is in radians. Note, that the vector elements are numbered from 2 to N where N is the number of buses in the power system and 1 is omitted because it would refer to the slack bus (or the main grid bus) for which the voltage and voltage angle are already known (1.0 + j0.0 p.u.).
    </p>
    <p>
      Next, the real and reactive power for a bus k for k = 2, 3, ...,N can be calculated according to the equations below utilizing the values for V and delta according the latest iteration's values for these variables. For the first iteration, these would be initial guesses of these values.
    </p>
    <figure>
      <img src="/images/power_flow_solution_NR/bus_real_reactive_power_eqns.png" class="centered">
      <figcaption class="centered"> Figure 2 - Real and reactive power for bus k.</figcaption>
    </figure>
    <p>
      The last major piece is the Jacobian matrix for the power system. It is shown below and split into for different regions, J1 to J4.
    </p>
    <figure>
      <img src="/images/power_flow_solution_NR/jacobian.png" class="centered">
      <figcaption class="centered"> Figure 3 - Jacobian matrix for a power system.</figcaption>
    </figure>
    <p>
      For instances where n does not equal k, J1 to J4 can be calculated as
    </p>
    <figure>
      <img src="/images/power_flow_solution_NR/jacobian_n_does_not_equal_k.png" class="centered">
      <figcaption class="centered"> Figure 4 - Jacobian matric formulas for n not equal to k.</figcaption>
    </figure>
    <p>
      and for instances where n does equal k, these equations can be used
    </p>
    <figure>
      <img src="/images/power_flow_solution_NR/jacobian_n_equals_k.png" class="centered">
      <figcaption class="centered"> Figure 5 - Jacobian matric formulas for n equal to k.</figcaption>
    </figure>
    <p>
      where k, n = 2, 3, ..., N.
    </p>

</section>

<span><br></span>

<section>
  <h2> The Code </h2>
    <p>
      A
    </p>
</section>

<span><br></span>

<section>
  <h2> The Test </h2>
  <p>
    L
  </p>
</section>

<span><br></span>

<section>
  <h2> Closing Remarks </h2>
    <p>
      I
    </p>

</section>
