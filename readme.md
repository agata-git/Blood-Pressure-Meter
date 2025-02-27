# Blood Pressure Meter

## Version
1.0.0

**ATTENTION**<br>
Device described here is student's experimental project. Do not rely on measurements taken by device like this one.

## Hardware design
![Block diagram](/Documentation/BloodPressureMeter_png.png "Block diagram")

## List of parts
- NUCLEO F446RE
- LCD Keypad Shield
- Honeywell Pressure Sensor ABPMANN005PG2A3
- KOGE mini pump KPM14A 3V
- KOGE electromechanical valve 3V
- Transistor N-channel BS170
- PNP transistor 2n2905
- four-way splitter, control aquarium valve, plastic tubes
- Solder breadboard, connectors 
- Hand cuff

## Photos
![Photo 1](/Documentation/IMG_4834.png)

![Photo 2](/Documentation/IMG_4839.png)

![Photo 3](/Documentation/IMG_4842.png)

![Photo 4](/Documentation/IMG_4807.png)

## Operation of the device
![Blood Pressure meter operation](/Documentation/WIN_20190808_152509.mp4)

## Measurement mechanism
During blood pressure measurement by blood pressure meter cuff placed on patient's arm
must be inflated till absolute closing of artery. Then valve is opened and cuff is slowly
deflated (approximetely 3-5 mmHg/s). When pressure in cuff equilibrates pressure in artery 
blood starts flow. This pressure can be considered as systolic pressure. When pressure in 
cuff drop below diastolic pressure value oscillation in cuff should disappear (what isn't 
entirely true as described later).
![NIBP](/Documentation/OscillometricMethodOfNIBP.png "Oscillometric method of NIBP measurement. Raw pressure signal from cuff and after bandpass filtration below [1]")
<div align="center"><em>
Oscillometric method of NIBP measurement. Raw pressure signal from cuff and after bandpass filtration below [1]
</em></div><br>
       
In this project 180 mmHg was assumed as pressure closing artery so the cuff was pumped up to 
180 mmHg. Then pump was turned off and valve was opened after 1 second delay in order to stabilize
pressure sensor output.

## Numerical calculations
Pressure sensor used in project has digital output so all operations on signal like
filtering must be performed digitally by microcontroller. All signals showed below were
computed by stm32f446re microcontroller and acquired with STM Studio. Sampling frequency 
of pressure signal from sensor equals 250 Hz.

### Systolic pressure evaluation
Systolic pressure was determined as value of first peak detected by sign change of first derivative 
of pressure signal. Since raw pressure signal contains noise which is significantly increased in 
differentiated signal, lowpass filtration is necessary. First choice was 3rd order IIR lowpass filter
with 20Hz cut-off frequency. This filter was insufficient - abrupt changes caused by valve opening
weren't damped enough as showed on a chart below. As a result that abrupt changes were detected as 
a first systolic peak.
![SYS](/Documentation/SYSmeasurementProblem.png)

Usage of 4th order IIR lowpass filter with 20Hz cut-off frequency eliminated this problem, as shown 
on a chart below.
![SYSflt](/Documentation/SYSmeasumrementfiltration.png)
![SYSlpflt](/Documentation/lowpassFiltered.png)

On another chart we can see peak, which was detected by applied method as systolic peak. 
![SYSpeak](/Documentation/detectedPeak.png "Detected systolic peak")
<div align="center"><em>
Detected systolic peak
</em></div><br>


### Diastolic pressure evaluation
Theoretically, when pressure in cuff drop below diastolic pressure value oscillation in cuff should disappear.
In fact, pulsatile blood flow still cause oscillations in partially air filled cuff. In order to detect diastolic pressure
using filtered pressure signal more sophisticated procedures like computing pulse wave parameters is needed. 

![Osc](/Documentation/cuffOscillations.png "Cuff pressure and amplified cuff pressure oscillations. So = cuff pressure for sudden increase in cuff pressure oscillations. As, Am and Ad are amplitudes of cuff pressure oscillations at systolic, mean and diastolic pressures, respectively. [2]") 
<div align="center"><em>
Cuff pressure and amplified cuff pressure oscillations. So = cuff pressure for sudden increase in cuff pressure oscillations. As, Am and Ad are amplitudes of cuff pressure oscillations at systolic, mean and diastolic pressures, respectively. [2]
</em></div><br>

Therefore diastolic blood pressure was calculated in simplified way, based on Systolic and MAP pressure calculations.
A common method used to estimate the MAP is the following formula [3]: 
```
MAP = DIA + 1/3(SYS – DIA)
```
Where: 
```
*   MAP - mean arterial pressure 
*   SYS - systolic pressure 
*   DIA - diastolic pressure
```

Based on this formula DIA pressure was calculated as follows:
```
DIA = (3*MAP - SYS)/2
```	
	
### Arterial Pressure evaluation
Mean Arterial Pressure was determined based on pressure signal after bandpass filtration. One of the 
the problems was choice of appropriate bandpass filter. Two filters were tested: FIR filter with lower 
cut-off frequency = 2 Hz and upper cut-off frequency = 10 Hz and IIR bandpass filter with the same cut-off 
frequencies. Since FIR filter introduces a delay, the output signal is shifted in time with respect to the input.
Group delay of applied filter is 83 so value of pressure signal 83 samples from before detected highest peak 
was considered as value corresponded to highest peak bandpass filtered signal.	

![MAP](/Documentation/signalsMAPcalculating.png "Highest peaks detected in bandpassed signals and corresponding pressure values: 5658=111mmHg and 5399=69mmHg") 	
<div align="center"><em>
Highest peaks detected in bandpassed signals and corresponding pressure values: 5658=111mmHg and 5399=69mmHg
</em></div><br>

As shown in the tables below values of MAP received by this two bndpass filtered signals in most cases differ 
from each other. MAP values are similar or even identical only when body and respiratory movement during examination 
were reduced practically to zero. Filtration by IIR filter was probably more sensitive on patient movements. 
Besides, results gained from signal filtered by IIR filter were in most cases lower 
and gave really low diastolic pressure values. Furthermore, signal after filtration by IIR bandpass filter is distorted
in comparison to signal after FIR filtration. Signal after FIR bandpass filter is more close to what pulse wave waveform
looks like. This effect is caused by unequal group delay, nonlinear phase and fact, that computations were perform on 
single-precision floating-point numbers[4]. IIR filters are very fast and effective but need for resolution may be critical in 
some cases, particularly with digital filters, when small but significant numbers combine with large numbers [5]. Efficiency of 
IIR filters in comparison to FIR filters results from feedback which requires appropriate precision of computations.	Therefore 
FIR filter was chosen.
Measurements in table below were performed on one person within one hour. Table contains values of MAP and diastolic 
pressure computed using signal filtered with FIR bandpass filter and MAP and diastolic pressure computed 
using pressure signal filtered with IIR bandpass filter as well.	

<table>
	<tr>
		<th> </th>
		<th align="center" colspan="2"><b>Calculations based on FIR filtered signal</b></th>
		<th align="center" colspan="2"><b>Calculations based on IIR filtered signal</b></th>
	</tr>
	<tr>
		<th align="center"><b>SYS</b></th>
		<th align="center"><b>MAP</b></th>
		<th align="center"><b>DIA</b></th>
		<th align="center"><b>MAP</b></th>
		<th align="center"><b>DIA</b></th>
	</tr>
	<tr>
		<td align="center">106</th>
		<td align="center">78</th>
		<td align="center">64</th>
		<td align="center">73</th>
		<td align="center">57</th>
	</tr>
	<tr>
		<td align="center">107</td>
		<td align="center">81</td>
		<td align="center">68</td>
		<td align="center">75</td>
		<td align="center">59</td>
	</tr>
	<tr>
		<td align="center">111</td>
		<td align="center">80</td>
		<td align="center">64</td>
		<td align="center">69</td>
		<td align="center">48</td>
	</tr>
	<tr>
		<td align="center">112</td>
		<td align="center">79</td>
		<td align="center">62</td>
		<td align="center">74</td>
		<td align="center">55</td>
	</tr>
	<tr>
		<td align="center">112</td>
		<td align="center">77</td>
		<td align="center">59</td>
		<td align="center">69</td>
		<td align="center">48</td>
	</tr>
	<tr>
		<td align="center">107</td>
		<td align="center">78</td>
		<td align="center">63</td>
		<td align="center">78</td>
		<td align="center">63</td>
	</tr>
	<tr>
		<td align="center">113</td>
		<td align="center">85</td>
		<td align="center">72</td>
		<td align="center">72</td>
		<td align="center">52</td>
	</tr>
	<tr>
		<td align="center">104</td>
		<td align="center">79</td>
		<td align="center">67</td>
		<td align="center">79</td>
		<td align="center">67</td>
	</tr>
	<tr>
		<td align="center">113</td>
		<td align="center">87</td>
		<td align="center">74</td>
		<td align="center">74</td>
		<td align="center">55</td>
	</tr>
	<tr>
		<td align="center">112</td>
		<td align="center">84</td>
		<td align="center">70</td>
		<td align="center">73</td>
		<td align="center">54</td>
	</tr>
</table><br>


## Blood pressure chart
![BPchart](/Documentation/blood-pressure-chart.png "Blood pressure chart [6]") 
<div align="center"><em>
Blood pressure chart [6]
</em></div><br>

## Author
Agata Pawlowska
agata.pawlowska07@gmail.com
 
## Licensing
GNU Public License 
 
## References
1. PETRIE, J. C., O'BRIEN, E. T., LITTLER, W. A. and D~
   SwErr, M. (1986): 'British Hypertensive Society: Recommendations
   on blood pressure measurements', Br. Med. J , 293,
   pp. 611-615
2. Leslie Alexander Geddes, M H Voelz, Christopher L. Combs, David S. Reiner, Charles F. Babbs: 
   Characterization of the oscillometric method for measuring indirect blood pressure, DOI:10.1007/BF02367308, 
   Published in Annals of Biomedical Engineering 1982  
3. https://www.ncbi.nlm.nih.gov/books/NBK538226/
4. Steven W. Smith: Digital Signal Processing - A Practical Guide for Engineers and Scientists, 2002
5. https://www.rane.com/note157.html
6. https://universityhealthnews.com/daily/heart-health/blood-pressure-chart-where-do-your-numbers-fit/
 