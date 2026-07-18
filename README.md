# Smart Home Tiny Leak Detection & Shutoff w/o Impeller

- Detect leaks as small as 5 cups using adaptive threshhold software
  
- This project starts with Yang's project at https://github.com/heyitsyang/Whole-House-Water-Leak-Controller .
	- Components
   
 		- Home Assistant (Scroll to the bottom to view screen shots)
     
		- Zigbee leak sensors
    
 		- Valve Controller communicates with Home Asistant using MQTT regarding
   			- Water Pressure Sensor
			- Motorized Valve
       		
- Optional Valve Controller adaptations
   
	- Buck converter may be replaced by dedicated 5v usb supply wall wart for ESP8266.
    
  	- Standard enclosure may be substituted for custom 3D printed electronics enclosure.
      
![Install](media/Installation.jpg)
    
- Additions for adaptive threshhold implementation
  
	- Motion detectors in kitchen and bathrooms communicate via MQTT
   
 	- Home Assistant and Valve Controller software adjustments
    
## Why Bother?

- Detect degraded plumbing joints that cause undetected leaks behind walls

- Friend who purchased house with prior insurance claim for leak behind wall was denied insurance
 
- Impellers corrode and stick over time; added installation and maintenance expense

- Detect faucets that weren't completely closed

- Water damage is expensive, inconvenient, and leads to future home insurance problems

## Observations

- Typical 24 hour pressure profile:
  
 ![Day](media/20240108_DAY_Plot.jpg)

  - Water pressure varies several psi during the course of a day
  	
  - Water pressure spikes when water heater is active
  	
  - Water pressure recovers slowly after demand if there is a pressure regulator

  - Water flow may not be detected by a static threshhold for slower flow rates or during periods of higher pressure

  - Somewhere between 65 - 66 psi is the lowest hard threshhold to indicate flow, but it may not be adequate in all scenarios
  	  
## How small a leak can we detect without using an impeller?

- Water flow duration was measured under various conditions (flush, shower, laundry, dishes, etc.)
  
	- An adaptive threshhold was devised to close the main valve 10 minutes after detecting water flow.
   
 	- Motion detected in the bathrooms or kitchen cause a 20 minute grace period (for showers, proof of consciousness, etc.).
  
  	- Leak vol = leak rate x flow detection time + 10 min x leak rate + water residue after valve closes (approx 20 fl oz = 625 ml)
  	   
 	- 75 ml/min flow is detected in 80 seconds. Total leak = 75(80/60) + 10(75) + 625 = 1475 ml (=6 cups).
  
        - 40 ml/min flow is detected in 6 minutes.  Total leak = 6(40) +10(40) + 625 = 1265 ml (=5 cups).
          
       	- 24 ml/min flow (22 drips/10 sec) is detected in 11 minutes.  Total leak = 11(24) + 10(24) + 625 = 1129 ml (<5 cups).
       	  
  	- Wide open faucet 5.3 liter/min flow would take 10 minutes to shut down.  Leak = 10(5.3)+.625 = 54 liters (14 gal).
  	 	- Therefore use moisture sensors to close valve immediately for larger volumes.
  	    
## Static pressure test results:

- Typical nightly results range betwwen -.03 and -.15 psi at my house.

- Pressure decrease is greater (-. 38 psi +-) if the test is run soon after the water heater has been activated and the pressure tank has peaked and is on the downswing. 

- A 24 ml/min leak causes a 3.44 psi drop during the nightly static pressure test under normal conditions.  This will get your attention!!
    
## Adaptive Threshhold Theory of Operation

- The adaptive threshhold is based on a "leaky peak detector".
  
  	- Leaky peak - If pressure (P) rises above the peak value, the calculated peak pressure is set equal P.
  	  
  		- The peak pressure stays constant for the next 10 minutes unless P exceeds it before then.
  	   
  	 	- After 10 minutes, the peak is reduced .25 psi if it doesn't go less than P.
  	    
  		- If P rate of decrease exceeds .25psi/10 min, the calculated peak pressure decreases at a slower rate.  Hence, the term leaky peak.
  	   
  	- Threshhold (T) - The threshhold that defines flow is set at 1 psi below the most recent calculated leaky peak pressure.
  	  
  	- Flow - When P goes below the threshhold, the 10 minute flow timer begins.  After 10 minutes, the valve is closed.
  	  
  	- Motion Sensors reset the flow timer when motion is detected.
  	  
  	- There is a similar mechanism to determine the threshhold when the flow stops and the pressure rises again.
  
  - Examples:
  	
   	- The peak and threshhold (T) are recalculated each time the pressure increases above the prior peak.

	  
	  ![Increase](media/20240104_165739%20Home%20p%20Incr%20plot.jpg)


	- The following plot shows the pressure decreasing at a rate of .125 psi/10 min; the peak is reduced .25 psi / 10 min until the two meet.

  	- The plots above and below demonstrate an example of the threshold tracking the normal ebb and flow of psi without tripping the flow threshld.

	  ![Decrease](media/20240104_174717%20G%20p%20decr.jpg)

   	- Adaptive Threshold operation through two demand cycles:
       
	- We see that at 8:11 and 8:27 when pressure drops below threshold (flow begins) the threshold flips to above the psi to define when flow stops.
 
   	- At around 8:14 and 8:33 the psi goes above the threshold, (flow stops), and the threshhold once again tracks below the newest max psi.
      
	![Adaptive2](media/AdaptiveThreshhold2.jpg)
 
## System Operation

- Based on appliance signatures, a 10 minute flow causes the valve to shut down unless the following conditions exist:
  
	- Motion is detected in the bathroom or kitchens, causing a 20 minute standby period.
   
	- The manual override switch has been actuated.
   
	- Detection of moisture by external sensors cause immediate valve closure.
   
 	- Monitor daily leakage test and graphical representation to catch other issues.

## Flow Duration Study

- Overnight activity followed by shower at 6 am.
  
	- Note visual sign of failing toilet seal, which was seen before heard:
      
![Toilet Flush](media/ToiLeak.jpg)

- This is a great visual tool to monitor the water system.  That seal is getting worse!  Still can not hear it to identify it. 

![Shower](media/ToiLeak2.jpg)

- Wash Machine flow and dishwasher signatures indicate max flow duration is only a few minutes.
  
	- Laundry:

![LaundrySignature](media/LaundrySignature.jpg)

	- Dishwasher:

![Dishwasher](media/DishwasherSignature.jpg)

- Morning routine including shower:

![Adaptive](media/Morning_routine_incl_shower.jpg)


## Hardware (Optional)

- PIR motion sensors with optional temperature/humidity report to Home Assistant, causing a 20 minute standby.
  
- Be sure to use a 100nf decoupling capacitor across temp sensor Vcc and Gnd to prevent spurious PIR hits.
  
- Use most any ESPxyz here.  Standardizing with ESP32 keeps my life simple, if inelegant.
  
- Add a white AILKIN USB power adapter and some velcro tape to the assembly and plug it in.
 
	- The blue LED is easily visible when motion is detected.
   
- Software for the PIR sensor is at https://github.com/Bobbo117/Cellular-IoT-Monitor/blob/main/src/AmbientAP/AmbientAP.ino .
 
	-  Enable HA so that it will communicate with home assistant or an alternative destnation.
   
	-  Use CASA_1 ID 4 (kitchen), 5 (bathroom), and 6 (bathroom2) for up to three PIR sensors.
   
   	-  These IDs will send the topics kithcen/pir, bathroom/pir, and bathroom2/pir respectively.
 	
![PIR_Disassembled](media/PIRDissassembly.jpg)

![PIR_Assembled](media/PIR_Motion_Detector.jpg)

## Software 

 - Adjustments to the original Watermain software are minimal:
   
	- New variable definitions and mqtt topics are appended at the beginning.
   
  	- New command topics are appended to the mqtt callback function.
     
	- New processing is appended at the end of the loop() function.
   
	- Two new threshold functions are appended at the end of the code after the loop function.

## Home Assistant Screens

- These screens along with Yang's will help you navigate your way through the software:

![Increase](media/20240104_165739%20Home%20p%20Incr.jpg)

![1](media/20240108_HA_Plot_PIR.jpg)

![2](media/20240108_HA_Settings.jpg)

![4](media/20240108_HA_full.jpg)

## Further Discussion

- If you take a long shower (> 10 minutes), verify that the bathroom motion sensor sees you!  Look for the blue light.
  
- Activate the Manual Override switch in the Home Assistant control screen for the powerwash vendor or other vendors using water.

- You can forgo the PIR motion sensors if you simply turn off the water momentarily before 10 minutes is up.

## Results

- This system has operated from March 2023 thru June 8, 2025 (today) with ONE unanticipated shutoff caused when the incoming pressure from the street dipped below 65psi, the absolute minimum threshhold.
   
- Very slow leak rates may not close the valve, especially if the pressure happens to be on the up cycle (increasing as would be expected during the routine ebb and flow).
  
   	- In this case, the nightly Static Pressure Test gives an abnormally high reading that should be followed up.

- Recently the Static Pressure Test gave excessive readings several nights in a row. This led to discovery that an outside faucet had not been closed tightly. There was no visible flow, just an occasional drip.

- It turns out that the PIR motion detectors are unnecessary at our house, because we have no continous flows close to 10 minutes.  Appliances don't draw water more than a few minutes at a time, and there are no long showerers.

- This system provides an added way to detect impending problems like the leaking toilet seal described above.  The graphic dispaly of psi over time adds a new dimension.  

## Conclusions

- While this adaptive threshold mechanism is not as sensitive as an impeller, it provides an added layer of protection without the installation and maintenance expense.
  
- This is a great project for those who want to become as one with their plumbing system while simultaneously protecting against one of the more frequent and expensive insurance claims. 












