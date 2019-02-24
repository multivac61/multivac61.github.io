+++
title = "Digitizing analog circuits containing op amps using Wave Digital Filters"
author = ["Olafur Bogason"]
date = 2016-03-20
lastmod = 2019-02-24T17:33:45+00:00
tags = ["dsp"]
categories = ["dsp"]
draft = false
+++

In this post I will share some work I have been doing on Wave Digital Filters, or WDFs for short. WDFs allows one to digitize analog reference circuits in a way that retains the underlying topology, has nice numerical properties, allows for breaking up of annoying delay-free loops when digitizing and has a very nice modular way of dealing with non-linearities. I will not explain the basic WDF theory here but if you are interested the [omnipotent paper](http://www.eit.lth.se/fileadmin/eit/courses/eit085f/Fettweis%5FWave%5FDigital%5FFilters%5FTheory%5Fand%5FPractice%5FIEEE%5FProc%5F1986%5F-%5FThis%5Fis%5Fa%5Freal%5Fchallange.pdf) on the subject written by Fettweis, the creator of WDF, or [this tutorial](https://ccrma.stanford.edu/~dtyeh/papers/wdftutorial.pdf) on WDF should get you familiar with the topic.

Until very recently the WDF formalism has only worked on reference circuits that can be decomposed into parallel or series sub-circuits. Since many circuits in the wild have much more complicated topologies, the scope of reference circuits available for digital modelling using WDF has been very limited. Late last year there was a paper published called [Wave Digital Filter Adaptors for Arbitraty Topologies and Multiport Linear Elements](https://www.ntnu.edu/documents/1001201110/1266017954/DAFx-15%5Fsubmission%5F53.pdf/a559ce90-d16b-49a3-a267-5b877d7fe70b). In it this issue was addressed by showing how arbitrary topologies may be handled within the WDF formalism. That was achieved by some cleaver usage of the ubiquitous [MNA method](https://en.wikipedia.org/wiki/Modified%5Fnodal%5Fanalysis). In essence the method starts out with a reference circuit, extracts all series/parallel sub-circuits and uses the MNA method on the remaining components and connections. The trick is to allow active elements to clump up inside the non series/parallel adaptors, commonly known as R-type (rigid) adaptors and then figure out how the outcoming waves depend on the incoming ones (scattering matrix).

Now I will give two examples of how to use this new method on circuits previously unobtainable under the WDF formalism. To warm up I will start off by coming up with a WDF structure for the [buffer amplifier](https://en.wikipedia.org/wiki/Buffer%5Famplifier).


## Op amp buffer circuit {#op-amp-buffer-circuit}

Following the steps developed in Wave Digital Filter Adaptors for Arbitraty Topologies.. I start out with a reference circuit, then approximate the op amp using a simple op amp model. I use a infinitely large resistor between the input poles so that I can use it when adapting for the port facing the voltage source (the only non-linear element that needs adapting).

<style>.org-center { margin-left: auto; margin-right: auto; text-align: center; }</style>

<div class="org-center">
  <div></div>

![](/images/wdf_buffer1.png)
![](/images/wdf_buffer2.png)
![](/images/wdf_buffer1_approx.png)

</div>

The next step is to form a so called replacement graph and find split components within it. The series/parallel adaptors that I find I remove from the graph and the remaining connections will fall inside the R-type adaptor we have to derive (it is impossible to further decompose the connections into more series/parallel connectors).

<style>.org-center { margin-left: auto; margin-right: auto; text-align: center; }</style>

<div class="org-center">
  <div></div>

{{< figure src="/images/wdf_buffer3.png" >}}

</div>

Now that we have the two WDF adaptors (series S, and a rigid one R) found in the approximated reference circuit we can find how incoming/outgoing waves are reflected when they reach the R-type adaptor. We do that by finding the scattering matrix which is obtainable by using [Modified Nodal Analysis](http://www.swarthmore.edu/NatSci/echeeve1/Ref/mna/MNA5.html). I chose to adapt the scattering matrix to the input voltage source instead of using a resistive voltage source for simplicity's sake.

<style>.org-center { margin-left: auto; margin-right: auto; text-align: center; }</style>

<div class="org-center">
  <div></div>

{{< figure src="/images/wdf_buffer4.png" >}}

</div>

This is the underlying WDF structure.

<style>.org-center { margin-left: auto; margin-right: auto; text-align: center; }</style>

<div class="org-center">
  <div></div>

{{< figure src="/images/wdf_buffer5.png" >}}

</div>

Place a voltage source and resistor on all ports and then populate the MNA matrix.

<style>.org-center { margin-left: auto; margin-right: auto; text-align: center; }</style>

<div class="org-center">
  <div></div>

{{< figure src="/images/wdf_buffer6.png" >}}

</div>

Modified Nodal Analysis matrix which we can use to figure out the scattering matrix.

<style>.org-center { margin-left: auto; margin-right: auto; text-align: center; }</style>

<div class="org-center">
  <div></div>

{{< figure src="/images/wdf_buffer7.png" >}}

</div>

Here is the SPQR tree that indicates how the computation of the WDF structure can be done. In each iteration the waves travel from the lowest part of the tree all the way to the top and then back after the input signal has been injected.


### Software implementation {#software-implementation}

Next I implemented the WDF model in Matlab code along with a simple LTspice simulation. Then I plotted the frequency response..

<style>.org-center { margin-left: auto; margin-right: auto; text-align: center; }</style>

<div class="org-center">
  <div></div>

{{< figure src="/images/wdf_volt_follower.png" >}}

</div>

As expected the buffer circuit holds the input at unity for all frequencies and the frequency response is visually not different from the ground truth LTspice simulation.


## Sallen-Key low pass filter {#sallen-key-low-pass-filter}

Next I move on to a bit more interesting reference circuit, the [lowpass Sallen-Key filter](https://en.wikipedia.org/wiki/Sallen%E2%80%93Key%5Ftopology#Application:%5FLow-pass%5Ffilter). The steps are exactly the same as above.

<style>.org-center { margin-left: auto; margin-right: auto; text-align: center; }</style>

<div class="org-center">
  <div></div>

{{< figure src="/images/wdf_sk1.png" >}}

</div>

We start out with a reference circuit..

<style>.org-center { margin-left: auto; margin-right: auto; text-align: center; }</style>

<div class="org-center">
  <div></div>

{{< figure src="/images/wdf_sk2.png" >}}

</div>

Approximate the op amp like before..

<style>.org-center { margin-left: auto; margin-right: auto; text-align: center; }</style>

<div class="org-center">
  <div></div>

{{< figure src="/images/wdf_sk_spqr.png" >}}

</div>

Generate the reference graph and find split components (again one series and one R-type adaptor)...

<style>.org-center { margin-left: auto; margin-right: auto; text-align: center; }</style>

<div class="org-center">
  <div></div>

![](/images/wdf_sk5.png)
![](/images/wdf_sk4.png)
![](/images/wdf_buffer6-1.png)

</div>

Populate the MNA matrix and write out the SPQR tree.

<style>.org-center { margin-left: auto; margin-right: auto; text-align: center; }</style>

<div class="org-center">
  <div></div>

{{< figure src="/images/wdf_sk3.png" >}}

</div>


### Software implementation {#software-implementation}

Again I coded up the WDF strucute and compared its frequency response with frequency response coming from LTspice.

<style>.org-center { margin-left: auto; margin-right: auto; text-align: center; }</style>

<div class="org-center">
  <div></div>

{{< figure src="/images/wdf_sallen_key_lp.png" >}}

</div>


## Demos {#demos}

Just to give a simple demo I put a funk drum beat and put it through the Sallen-Key lowpass filter WDF structure with a cutoff frequency of ~1 kHz.

<br>
<iframe width="100%" height="450" scrolling="no" frameborder="no" src="https://w.soundcloud.com/player/?url=https%3A//api.soundcloud.com/playlists/207645541&amp;auto_play=false&amp;hide_related=false&amp;show_comments=true&amp;show_user=true&amp;show_reposts=false&amp;visual=true"></iframe>


## Final thoughts {#final-thoughts}

There are still many circuits that are impossible to model using state-of-the-art WDF theory. An example are circuits that have global feedback (such as the MS-20 filter). Fundamental research in the field is active at the moment and hopefully within a few years those circuits will also be able to model using WDFs.

A big shout out goes to [Kurt James Werner](https://ccrma.stanford.edu/~kwerner/) for help and support down the WDF rabbit hole.
