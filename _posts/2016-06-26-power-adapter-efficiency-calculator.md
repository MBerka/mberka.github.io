---
layout: post
title: "Power supply efficiency calculator"
date: 2016-06-26 13:17:00 -0600
categories: electronics
---
**How much power is your charger or adapter using?**

The true electrical efficiency of a power supply can only be determined by measuring the average input and output power over a period of time. Lacking time or equipment, one can only guess based on the device label. For example:

INPUT: 100-240 V ~ 50-60 Hz, 0.2A

OUTPUT: 5.0 V ⎓ 550 mA

While the output power can be taken at face value (the device must be able to supply the displayed voltage and current, with a product of 2.75 W),
input power should not. Even with 100 V mains power, that would imply an average power of 70.7 V * 0.14 A = 10 W (note sine wave root-mean-square), leading to an efficiency of under 28%. In many cases, far better is required.

## <a name="TOC-Efficiency-regulations"></a>Efficiency regulations
In response to once-abysmal efficiencies, various authorities placed limits on the power a commercial electrical adapter (external power supply) can waste when loaded (when current is drawn from it) and unloaded (when powered but unused, a state of 0% efficiency commonly known as "vampire power"). The [Energy Star levels for external power supplies](https://www.energystar.gov/index.cfm?c=archives.power_supplies) are currently used, with gradually increasing standards numbered with Roman numerals (currently III-VI). Higher-power devices must have greater efficiencies, which limits the total power wasted per device, and are allowed greater no-load power drain.


According to the EU [Review Study on Commission Regulation (EC) No. 278/2009, External Power Supplies](https://www.power.com/sites/default/files/greenroom/docs/EPSReviewStudy_DraftFinalReport.pdf), a new standard (VII?) that increases the required efficiency by 2.5% and decreases the maximum no-load power by 2.5% is proposed to take effect in 2019. These are small changes, but the standards mean that most chargers are already 70%+ efficient while charging, with newer models approaching 90% and maximum no-load power below 0.1 W for small chargers.

Current information on US/EU/Canadian regulations is [provided by CUI Inc.](http://www.cui.com/efficiency-standards), an electronics designer.

## <a name="TOC-Understand-your-rating"></a>Understand your rating
Since few chargers or power supplies list efficiency on the casing, these standards are frequently the only way to approximate it short of electrical testing. The following calculator determines which formulas to apply to find the thresholds the manufacturer had to meet in order to meet the standards. Note that the device may well have better values, especially if it claims the highest certification available at the time of production. It may also have slightly worse ones at some currents: the "average efficiency" that is compared to the thresholds calculated by the formulas is the mean of the efficiencies at 100%, 75%, 50%, and 25% of the rated output current. An above-bar value at one of these points could allow for a below-bar value at another. The closer the threshold is to 100%, the less possible variation there is.

### <a name="TOC-Instructions"></a>Instructions
Look for Roman numerals in a circle to make sure that this calculator applies.

For anything below V, you only need to enter the numerals and the power rating

(the number preceding "W" or "watts"). Later regulations consider more factors - low and high output voltages beginning in V, AC-AC converters and multiple output voltages in VI - and apply different formulas based on these.

### <a name="TOC-Calculator"></a>Calculator
<style>
  #right-button {
    cursor: pointer;
    width: auto;
    display: inline-block;
    border: #000 1px solid;
  }
  .b{
    font-weight:bold;
  }
  .disabled {
    color: gray;
  }
  input{
    width: 30px;
  }
  input\x5btype="checkbox"\x5d {
    width:13px;
    height:13px;
  }
  select {
    width: 40px;
  }
  #error {
    color: red;
  }
  #results span{
    font-weight:bold;
  }
</style>


<div class="items">

  <label><span class="b">Output: </span><input id="voltage" value="5"/> V</label>&nbsp;&#9107;
  <label><input id="current" value="1"/> A</label><br/>
  <label><span class="b">Roman numeral: </span>
    <select id="numeral">
      <option value="I">I</option>
      <option value="II">I</option>
      <option value="III">III</option>
      <option value="IV">IV</option>
      <option value="V" selected>V</option>
      <option value="VI">VI</option>
    </select>
  </label><br/>
  For VI or higher:<br/>
  <label><input type="checkbox" id="ac-ac" value="1">AC-AC (~ output)</label><br/>
  <label><input type="checkbox" id="mult" value="1">Can produce multiple output voltages simultaneously</label>
  <br/><br/>
  <div id="right-button">&gt; Calculate &lt;</div><br/>

  <span id="error"></span><br/><br/>
  <div id="results" class="disabled">
    Maximum power use when device not connected: <span id="vampire"></span><br/>
    Minimum efficiency when in use: <span id="efficiency"></span><br/><br/>

    Maximum output power when in use: <span id="power"></span><br/>
    Maximum power wasted when in use: <span id="waste"></span><br/>
    Maximum total power consumption when in use: <span id="total"></span>
  </div>
</div>

<script>
  var calculate_efficiency = function() {
    var voltage = document.getElementById("voltage").value;
    var current = document.getElementById("current").value;

    var error_message = "";
    if(isNaN(voltage) || isNaN(current)) {
      error_message = "Please enter decimal numbers in the voltage and current spaces";
    } else if(voltage < 0 || current < 0) {
      error_message = "Please enter positive numbers in the voltage and current spaces";
    }

    if(error_message == "") {
      var rated_power = voltage*current;
      var low_voltage = (rated_power < 6 && current >= 0.55);

      var roman_numeral = document.getElementById("numeral").value;

      if(rated_power > 250) {
        if(roman_numeral == "III" || roman_numeral == "IV" || roman_numeral == "V") {
          error_message = "Power out of range (early standards only up to 250 W)";
        }
      }
    }
    if(error_message != "") {
      document.getElementById("error").innerHTML = error_message;
      return;
    }

    var ac_ac = document.getElementById("ac-ac").checked;
    var mult_voltage = document.getElementById("mult").checked;
    var vampire = "";
    var eff = "";

    switch(roman_numeral) {
      case "I":
        eff = rated_power;
        vampire = 0;
        break;
      case "II":
        eff = rated_power;
        vampire = 0;
        break;
      case "III":
        if(rated_power <= 10) {
          vampire = 0.5;
        } else {
          vampire = 0.75;
        }

        if(rated_power <= 1) {
          eff = rated_power * 0.49;
        } else if(rated_power <= 49) {
          eff = 0.09 * Math.log(rated_power) + 0.49;
        } else {
          eff = 0.84;
        }

        break;
      case "IV":
        vampire = 0.5;

        if(rated_power <= 1) {
          eff = rated_power * 0.5;
        } else if(rated_power <= 51) {
          eff = 0.09 * Math.log(rated_power) + 0.5;
        } else {
          eff = 0.85;
        }
        break;
      case "V":
        if(rated_power < 50) {
       vampire = 0.3;
          if(rated_power <= 1) {
            if(low_voltage) {
              eff = 0.48 * rated_power + 0.14;
            } else {
              eff = 0.497 * rated_power + 0.067;
            }
          } else {
            if(low_voltage) {
              eff = 0.0626 * Math.log(rated_power) + 0.622;
            } else {
              eff = 0.075 * Math.log(rated_power) + 0.561;
            }
          }
        } else {
          vampire = 0.5;

          if(low_voltage) {
            eff = 0.87;
          } else {
            eff = 0.86;
          }
        }
        break;
      case "VI":
        if(mult_voltage) {
          vampire = 0.3;
          if(rated_power <= 1) {
            eff = 0.497 * rated_power + 0.067;
          } else if(rated_power <= 49) {
            eff = 0.075 * Math.log(rated_power) + 0.561;
          } else {
            eff = 0.86;
          }
        } else {
          if(rated_power > 250) {
            vampire = 0.5;
            eff = 0.875;
          } else {
            if(!ac_ac && rated_power <= 49) vampire = 0.1;
            else vampire = 0.21;

            if(!low_voltage) {
              if(rated_power <= 1) {
                eff = 0.5 * rated_power + 0.16;
              } else if(rated_power <= 49) {
                eff = 0.071 * Math.log(rated_power) + 0.67;
              } else {
                eff = 0.88;
              }
            } else {
              if(rated_power <= 1) {
                eff = 0.517 * rated_power + 0.087;
              } else if(rated_power <= 49) {
                eff = 0.0834 * Math.log(rated_power) - 0.0014 * rated_power + 0.609;
              } else {
                eff = 0.87;
              }
            }
          }
        }
        break;
      case "VII":
      case "VIII":
      default:
        vampire = rated_power;
        eff = 0;
    }

    document.getElementById("power").innerHTML = rated_power.toFixed(2) + " W";
    document.getElementById("vampire").innerHTML = vampire.toFixed(2) + " W";
    document.getElementById("efficiency").innerHTML = (100*eff).toFixed(1) + "%";
    document.getElementById("waste").innerHTML = (rated_power / eff - rated_power).toFixed(2) + " W";
    document.getElementById("total").innerHTML = (rated_power / eff).toFixed(2) + " W";

    document.getElementById("results").className = "enabled";
  };

  var submitButton = document.getElementById("right-button");
  submitButton.addEventListener("click", calculate_efficiency);
  submitButton.style.fontWeight = "bold";
  submitButton.style.backgroundColor = "cyan";

</script>
