1. OVERVIEW

    1.1 Purpose of the BSPD

        The Brake System Plausibility Device (BSPD) is a non programmable, indipendent safety circuit, which monitors the driver's brake pressure and power demand at the same time.
        If occours a situation in which the driver requires high motor power(>5kW) and hard braking(> 30bar) simultaneously for more than 500ms, the device opens the Shutdown Circuit(SDC), removing power from the motor.
        The fault must be latched: if the conditions return to normal, the SDC must remain open, and it can only be reset byshutting down and re-powering the device(No reset buttons according to the task).
        The device also monitors the integrity of both sensors, if one of them gives an out of range reading, the device opens immediately the SDC, without waiting for the timer.
        The design uses only analog and discrete logic components, with no microcontrollers or programmable logic.


    1.2 Design principles

        -Fail-safe output: 
            The relay is powered only while all the components are working correctly. Any loss of power, broken wires or component failure means removing power to the relay, which opens the SDC.

        -Single signal convention:
            From the time stage onward every logic signal is intended "ok" when high and "fault" when low, to guarantee the fail-safe output.

        -Ratiometric thresholds:
            Sensor and thresholds share the same filtered 5V supply, so small variations affect signal and threshold equally and do not shift the measurement.

        -Separated Supplies:
            The relay coil are fed through a different 5v branch, in order to keeo the raly and switching noise away from the comparator threshold.

    1.3 Block diagram

        ![BSPD BLOCK DIAGRAM](/img/BSPD_block_diagram.JPEG)
        
        The signal flows from top to bottom: The sensors enter throug the input filters, are converted into logic levels in the analog section, are evaluated in the logic section and command the relay in the output section.        

2. DESCRIPTION OF THE BLOCKS
    2.1 Power
        -Requirements: The power supply block is asked to take an unstable voltage between 9V and 17V and regulate it into a 5V to power the device.

        -Block Overview: [connector] -> [protections] -> [buck] -> [5V traces] -> [device's components].

        -Input protection:The first protection is a Schotty diode against reversed polarity than a TVS diode is placed before the switching buck in order to protect it from high-voltage spikes. After it there are 2 capacitors, the 100uF is needed to avoid voltage drops at a low frequency, the 4.7 uF is needed to cover high frequency voltage drops.

        -Current budget: To projecting the power supply an estimated current consumption was needed.
        Sensors (from project requirements): 30mA
        Voltage dividers (I = V/R, see section 2.3): 1.5mA
        Comparators (2 × TLV9024-Q1) and output pull-ups: 2.3mA
        Logic block: 1mA
        LEDs (5 × 2 mA): 10mA
        Relay coil: 28mA
        Total: 75mA

        -Regulatore dissipation
        The board drwas about 75mA, which is 10% of the buck's rated current, leavil a big margin for thermal derating, startup inrush and potential future changes.
        With P_out = 5V x 75mA = 0,38W, with an 85% effincency at 12V(from the datasheet), P_in will be about 0.44W.
        P_loss = P_in - P_out = 0,44W - 0,38W = 0,06W.
        The schotty diode adds about 50mA * 0,3V = 15mW. Both values are very small, so no heat sinking is needed
        
        -Regulator choices:
        The switching buck was chosen in order to have a better thermal efficency, the LDO would have had too much power to dissipate and the buck + LDO design was overkilled. (see decision.md)
        The ADI LT8607EMSE#PBF  was specifically chosen sinche it had a big margin for voltage spike and output current, Wide frequency range(200 kHz-2,2MHz).   
        The frequency was set a 2MHz, which is above the AM band. 
        Due to the low current the buck works in burst mode, but still guaraantees a ripple <= 10mV, therefore there is no filter was added on the analog 5V supply, since the comparator hysteresis is 50mV.   

        -Calculation
        TVS diode: SMBJ18A-E3/5B, standoff tension 18V, right above the 17V max limit, breakdown tension 20V and clamping tension 29,2V, well below the V_in max of the buck(42V).

        C1: Is placed to stop the input from ringing when the board is connected. To do this it must be much larger than c2(x4) and its resistance should be close to the value that most damps c2 and the cable.
        Assumin a meter long cable and conidering the loss of capacity in the cold, the best option was a 100uF 50V electrolitic capacitor, being comfortably above the minimum capacity and having a simular resistance.

        C2: X7R 4.7 µF / 50 V ceramic capacitor placed right next to the buck converter, and it handles the fast current pulses the converter draws while switching(capacity as in the data sheet).

        L: The value(4.7uH) was taken from the datasheet reference design. The peak current was calculate in the worst possible case, startup, when the capacitors on teh 5V rail are fully discharged. The result was 0.30A, the chosen inductor(Coilcraft XFL4020-472MEC) has a saturation current of 2.0A.

        C4: 10 nF soft-start capacitor, t_SS = 10 nF × 0.778 V / 2 µA ≈ 3.9 ms (datasheet p. 16). It limits the start-up current and is used in the inductor calculation.
    
        R2 & R3: The resistors values (1M & 187k) were taken from the datasheet design. Using the reference FB voltage (0.778V) the V_OUT will be about 4.94V, about 1.2% lower than the reference 5V, which is an acceptable error.

        C6: The requested minimum capacity was about 10uF, a 22uF x7R ceramic capacitor was chosen. The larger value is a direct consequence that thsi kind of capacitors lose part of the capacity with a DC voltage applied to it. With 22uF the calculated ripple(1.1mV) is well below the 10mV target.
        (C_BST = 0.1 µF, C_INTVCC = 1 µF and R_T = 18.2 kΩ were taken from the datasheet reference design.)

        FB1: A 600 Ω @ 100 MHz ferite bead was placed on the relay branch, so that the coil current step does not affect the rest of the 5V rail. The coil draws about 28mA, well below the 1.3A rating, and the voltage drop is only 4mV, the relay still gets 4.99V.

        -Relay coil supply: To prevent noise generated by the coil from disturbing the rest of the circuit, the relay coil is feed through a different branch of the regulated 5V rail, which is filtered.
        A ferrite bead(Murata BLM18KG601SH1D, 600 Ω at 100 MHz) divides this branch, while 2 capacitors placed near the relay coil complete the filter.
        A 10uF capacitor acts like locacl charge reservoir, to avoid voltage drops in the logic rail at turn on. And a 100nF capacitor redirects high frequency noise to the ground.

        -worst case analysis:
        Tolerances: resistor  ±1%
        5V rail, usint voltage partitioner, in the following situation R2_max & R3_min and R2_min & R3_max, the possible outcome is 4,76 to 5,15.


        -Simulation
        All the pictures of the simulation are in the [sim/power](../sim/power/) folder.


    2.2 Inputs
        -Requirements
        The input block must take in the sensor signal, with the background noise, and give a clean output without altering the power or brake signal. The filter delay must be unnoticable compared to the 500ms timer. The block must also make disconnected sensor detectable.
        -Block overview
        [sensors] -> [RC filter] -> [analog block]
        -Assumption
        The signals must chang in the tens of milliseconds range, while the noise picked up from the inverter is much faster(KHz to MHz).  The comparator inputs draw almost no current, so they do not lead the filter.
        -Calculation
        Given the assumptions, the filter is identacal for each sensor.
        The filter time costaant was set to 1ms, so that the filter can settle(at 5τ = 99,3%) in about 1% of the 500ms timer.
        With a 10k resistor and a standart 100nF capacitor, the cut-off frequency is about 158Hz, the fastest pedal movemete (about 50ms) corresponds to 7Hz.
        The filter lets the signal pass almost with no changes, 99.8% considering 10Hz. While the inverter nois at 10kHz is reduced of about 63 times.
        Attached to the pwr_in and brake_in wires there is a 100k pull down resistor to put the voltage to 0, making the disconnected sensor detected.

        -Simulation
        All the pictures of the simulation are in the [sim/inputs](../sim/inputs/) folder.


    2.3 Analog
        -Requirements
        The analog block must detect when the power and braking sensors are over their trigger threshold (5kW and 30 bar), and when the sensor give an out of range value, out of the 0,5-4,5V interval.
        The result are sent to the Logic block.
        -Block overview
        [Filtered signals] -> [comparators] -> [logic block]
        -Trigger threshold
        Brake threshold:
        The brake pressure sensor is linear: 0.5 V at 0 bar and 4.5 V at 100 bar.
        The sensor has sensitivity of (4,5-0,5)V/100bar 0.04V/bar. To find the 30 bar corresponding voltage, the sesitivity must be multiplied by the pressure and than added the 0.5V offset:
        0,5V + 0.04V/bar*30bar = 0,5V + 1,2V = 1,7V.
        The comparator must therefore activate when the signal rises aboe 1,7V.

        Power threshold:
        The power sensor is linear: 0,5V at -100A, 2,5V at 0A and 4,5V at 100A.
        For the calculation we can just operate with postive current, since the negative means the motor is giving current to the battery.
        The sensor's sensitivity is (4,5-2,5)V/100A = 0,02V/A
        The trigger power is 5kW, to find the current value we need to chose the right voltage value, which is in the 400V-600V range by the battery specs, by using A = P/V.
        Choosing 600V as reference:
        A = 5kW/600V = 8.33A, so the triggere voltage will be at 2,5V + 0,02V/A*8.33A = 2,5V + 0,167 = 2,667V.
        But in this case when the voltage drops to 400V, the reference power would decrease to 8,33A*400V = 3,33kW, letting the BSPD lock at a lower power.
        Choosing 400V as reference:
        A = 5kW/400V = 12.5A, so the trigger voltage would become 2,5V + 0,02V/A*12,5AA = 2,5V + 0,25V = 2,75V.
        But at the start of the race, when the battery delivers 600V the power would be 12,5A*600V = 7,5kW.

        Wanting to keep the max deliverable power at 5kw, the right decision is by choosing 600V as reference, so the power threshold will be 2,67V.        

        -Calculation
        For each voltage partitioner the sum of the resistors will be about 100k, in order to have an ignorable current, about 49uA.
        The ratio between R_S and R_H (10 kΩ / 1 MΩ) was chosen to obtain about 50 mV of hysteresis, about eight times the residual noise after the input filter. Since this ratio raises trigger point by 1 %, the divider voltages must be 1 % lower: 1,7/1,01 = 1,683V & 2,667/1,01 = 2.640V.

    2.4 Logic
        -Requirements
        The logic block must provide a single signal, which is high when evertyhing works and goes to low then something is wrong(HArd braking with high power for more than 500ms or sensor fault).
        Once low the signal must be low until the device is powered off.
        - Block overview    
        [BRAKE_HIGH, POWER_HIGH] -> [AND 1] -> [RC timer] -> [Schmitt inverter] -> [AND 2] -> [latch] -> [Output block]
                                                                      [SENSOR_OK] -^
        -Timer
        The timer is based on RC architecture, powered from the AND 1, and followed by a schmitt trigger inverter.
        When hard braking and high power are present at the same time, the AND1 output goes high the capacitor starts charging and when crosses the trigger'sthreshold, the inverter goes low(fault).
        The schmitt is placed because the capacitor charges slowly,and thanks to his hysteresis it switches only with a clean edge.
        The chosen schmitt triggere is the Nexperia 74HC14PW-Q100, which has positive going threshold of 2.38V when powerd at 4.5V and 3.14V at 6.0V. 
        Interpolating between the two at 4.94V the threshold will be about 2.60V.
        From inverting the capacitor charging formula t = 0,75τ, the chosen delay was 485ms, just below the 500ms, this gives a τ = 0.65s,
        Choosing a 1uF capacitor, and τ = R*C we got R = 650k.

        -Start up reset
        When turned on the latch could start in a random state, and the sensors filters need a few milliseconds to settle.
        With a 1uF capatitor and a 100k resistor, followed by a smith inverter, keeps ready low for about 117ms after power up(when the 1.54V_T- is crossed), holding latch in OK state. 

        -Simulation
         All the pictures of the simulation are in the [sim/logic](../sim/logic/) folder.

        
    2.5 Output
    2.6 Connectors
        All the external connections use connectors from the same family(Molex Micro-Fit 3.0). The 2 main features are the locking latch, which keep the cable connetced also under the vehicle vibrations, and a polarized housing, to prevent the connector to be inserted the wrong way. 
        The 2 pin version(43650-0201) is used for the power input and the 3 pin version(43650-0301) for the sensors. Since some connectors are identical, each cable must be labelled and colour-coded to avoid plugging it into the wrong connector.


