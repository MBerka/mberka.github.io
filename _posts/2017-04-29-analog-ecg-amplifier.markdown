---
layout: post
title:  "Analog ECG amplifier"
date:   2017-04-29 15:02:00 -0600
categories: electronics
---
Years ago, a team consisting of [Renata Rycyk](https://pl.linkedin.com/in/renata-rycyk-a91b28116), Sylwia Krysiak, and myself implemented a simple design of a one-channel amplifier for [electrocardiogram signals](https://en.wikipedia.org/wiki/Electrocardiography). I have posted our work to highlight the potential pitfalls and share the resulting files. To anyone taking up a similar project in the future - good luck!

The design discussed here addresses the following assumptions:
<table border="1" bordercolor="#888" cellspacing="0" style="border-collapse:collapse;border-color:rgb(136,136,136);border-width:1px"><tbody><tr><td style="width:139px;height:15px">**Property** </td><td style="width:205px;height:15px">**Value** </td></tr><tr><td style="width:139px;height:31px"> Input</td><td style="width:205px;height:31px"> Two-sided differential voltage </td></tr><tr><td style="width:139px;height:15px"> Frequency band</td><td style="width:205px;height:15px"> 0.05 - 250 Hz</td></tr><tr><td style="width:139px;height:15px"> QRS amplitude</td><td style="width:205px;height:15px"> 0.5 - 5 mV</td></tr><tr><td style="width:139px;height:15px"> Amplification</td><td style="width:205px;height:15px"> 1000 V/V (60 dB)</td></tr><tr><td style="width:139px;height:35px"> Desired elements</td><td style="width:205px;height:35px"> Battery supply
 Cable shielding</td></tr></tbody></table>
Initial simulations were carried out in LT SPICE. The following piecewise-linear signals were used to visually model an ECG signal for time-domain simulation:

<table border="1" bordercolor="#888" cellspacing="0" style="border-collapse:collapse;border-color:rgb(136,136,136);border-width:1px"><tbody><tr><td style="width:63px;height:15px"> Signal</td><td style="width:583px;height:15px"> Code</td></tr><tr><td style="width:63px;height:42px"> Full wave</td><td style="width:583px;height:42px"> PWL REPEAT FOR 5(0 0 80m 0 120m 1.5m 160m 0 240m 0 265m -2m 290m 5m 315m -2.5m 340m 0 440m 0 520m 2.5m 600m 0 625m 0m5 650m 0 860m 0)ENDREPEAT</td></tr><tr><td style="width:63px;height:31px"> Positive</td><td style="width:583px;height:31px"> PWL REPEAT FOR 5(0 0 80m 0 120m 1.5m 160m 0 272m 0 290m 5m 308m 0 340m 0 440m 0 520m 2.5m 600m 0 625m 0m5 650m 0 860m 0)ENDREPEAT</td></tr><tr><td style="width:63px;height:33px"> Negative</td><td style="width:583px;height:33px"> PWL REPEAT FOR 5(0 0 240m 0 265m -2m 272m 0 308m 0 315m -2.5m 340m 0 860m 0)ENDREPEAT</td></tr></tbody></table><div style="display:block;text-align:center;margin-right:auto;margin-left:auto"><a href="https://mberka.com/electronics/analog-ecg-amplifier/LTWaveformModel.jpg?attredirects=0" imageanchor="1"><img border="0" src="https://mberka.com/_/rsrc/1493510740618/electronics/analog-ecg-amplifier/LTWaveformModel.jpg" /></a>
<p style="text-align:center">*Full-wave signal model following amplification, centered around 2.5 V virtual ground.*</p>

While we initially looked at various designs involving a Right Leg Drive, these did not work for us in practice. Whether due to electrode contacts or differences in phase shift, noise was reduced more by connecting the leg to the circuit zero. This same potential was also applied to the shielding. To allow a (nearly) +/- 5V output potential while limiting battery volume, this zero was a virtual produced by a high-resistance voltage divider. In retrospect, this was overly simplistic, and the resulting "zero" was prone to change. I recommend **at least**  a [buffered ground](https://tangentsoft.net/elec/vgrounds.html).

While we initially thought it might be more economical to assemble an instrumentation amplifier from three operational amplifiers, the AD 621 in-amp chip provided better testability and consistency. As design was carried out using LT SPICE and an AD621 model was not readily available, simulations were carried out with an LT1167 model:

<div style="display:block;text-align:center;margin-right:auto;margin-left:auto"><img alt="Schematic used in simulation of amplifier stages" border="0" src="https://mberka.com/_/rsrc/1493508154637/electronics/analog-ecg-amplifier/SimulationSchematic.jpg" />
<p style="text-align:center">*(LT SPICE circuit attached)*</p></div>

The simulated results matched expectations across the frequency band:

<div style="display:block;text-align:center;margin-right:auto;margin-left:auto"><img alt="Graph of overall output amplification in modeled circuit." border="0" src="https://mberka.com/_/rsrc/1493508456825/electronics/analog-ecg-amplifier/OverallAmplificationSimulationGraph.jpg" /></div>

The circuit was breadboarded and the frequency characteristic of each filter and of the overall circuit was determined by sweeping three frequencies per decade with a ~1.4 mV sine wave. Input versus output were measured via oscilloscope. The circuit as a whole behaved as desired; we were fortunate that the concept of the instrumentation amplifier is universal enough that no problems resulted from the model substitution.

<div style="display:block;text-align:center;margin-right:auto;margin-left:auto"><img alt="Amplification [dB] vs. Frequency [Hz]" border="0" src="https://mberka.com/_/rsrc/1493502036439/electronics/analog-ecg-amplifier/CaloscLadna.JPG" /></div>
<p style="text-align:center">*(measurement spreadsheet with graphs for individual stages attached)*</p>

The circuit was not ideal in practice, in practice, but just able to output a recognizable signal under optimal circumstances:

<div style="display:block;text-align:center;margin-right:auto;margin-left:auto"><img border="0" src="https://mberka.com/_/rsrc/1493510743908/electronics/analog-ecg-amplifier/ScopeScreen.jpg" /></div>
<p style="text-align:center">*Amplified RL lead as shown by oscilloscope.*</p>

A power switch was added, along with a power indicator (lit on the right) to reduce waste (when the amplifier was not in use) and confusion as to whether the device was operating (when it was). Current through it was reduced to the smallest that would produce discernible light. The following circuit (switch not shown) was assembled:

<div style="display:block;text-align:center;margin-right:auto;margin-left:auto"><img alt="Schematic including power indicator" border="0" src="https://mberka.com/_/rsrc/1493506395531/electronics/analog-ecg-amplifier/FinalSchematic.JPG" style="width:100%" />

*(LT SPICE "View Only" file attached, along with the modified symbol file used to disguise the LT1167)*

As the circuit seemed too simple to merit a PCB, it was soldered to a copper-plated perforated board, which was in turn enclosed in a four-part plastic casing. Challenges included fitting everything and positioning a variety of components connected to both the board and the front plate of the casing while preventing short circuits. Isolation was provided by shrink tubing where possible, and by hot glue (found to be the non-conducting counterpart of solder) where not. Lesson learned: better too much wire than too little. A  few short wires close together are more trouble than a large number that can be spread out.

<div style="display:block;text-align:center;margin-right:auto;margin-left:auto"><img border="0" src="https://mberka.com/_/rsrc/1493504478040/electronics/analog-ecg-amplifier/GoraOtwarty.jpg" /></div>

<p style="text-align:center">*Top view of assembled circuit.*</p>

<div style="display:inline;margin:5px 10px;float:right"><img alt="Front of amplifier when running" border="0" src="https://mberka.com/_/rsrc/1493504481398/electronics/analog-ecg-amplifier/Przod.jpg" />
The power indicator (lit in the image to the right, off above and below) was adequate.

The input leads entered the casing as a pair on the left, apart from the right-leg cable (right). The output was provided as two wires;
a more elegant connection was considered, but the raw form allowed easier connection of the signal to a breadboard, scope, or other system.

<img alt="Top view with ECG leads" border="0" src="https://mberka.com/_/rsrc/1493504474858/electronics/analog-ecg-amplifier/GoraKable.jpg" style="border-width:1px;border-style:solid;border-color:rgb(204,204,204) rgb(153,153,153) rgb(153,153,153) rgb(204,204,204);padding:7px;color:rgb(0,0,204);font-size:13.3333px;display:block;margin-right:auto;margin-left:auto;text-align:center" />
<p style="text-align:center">*Top view of amplifier and cables.*</p>

End conclusion - if you have a choice between reading signals from the body and reading the output of a digital sensors - choose the latter.
