1. OVERVIEW

    1.1 Purpose of the BSPD

        The Brake System Plausibility Device (BSPD) is a non programmable, indipendent safety circuit, which monitors the driver's brake pressure and power demand at the same time.
        If occours a situation in which the driver requires high motor power(>5kw) and hard braking(> 30bar) simultaneously for more than 500ms, the device opens the Shutdown Circuit(SDC), removing power from the motor.
        The fault must be latched, if the conditions turn back to normal the SDC must remain open, it can be reset only by shutting down and re-powering the device(No reset buttons according to the task).
        The device monitors also the integrity of both sensors, if one of them gives an out of value reding the device open immediately the SDC, without the timer.
        The design uses only analog and discrete logic components, with no microcontrollers or programmable logic.


    1.2 Design principles

        -Fail-safe output: 
            The NO relay is powered only while all the components are working correctly. Any loss of power, broken wires or component failure means removing power to the relay which end in the SDC opening.

        -Single signal:
            From the time stage onward every logic signal is intended "ok" when high and "fault" when low, to garantee the fail-safe output.

        -Ratiometic thresholds:
            Sensor and thresholds share the same filtered 5V supply, so small variaton affect signal and threshold equally and do not shift the measurerment.

        -Separated Supplies:
            The relay coil are fed through a different 5v branch, in order to keeo the raly and switching noise away from the comparator threshold.

    1.3 Block diagram

        ![BSPD BLOCK DIAGRAM](/img/BSPD_block_diagram.JPEG)
        
        The signal flows from top to bottom: The sensors enter throug the input filters, are converted into logic levels in the analog section, are evaluated in the logic section and command the relay in the output section.        

2. DESCRIPTION OF THE BLOCKS
    2.1 Power
        -Requirements: The power supply block is asked to take an unstable voltage between 9V and 17V and regulate it into a 5V to power the device.

        -Block Overview: [connector] -> [protections] -> [buck] -> [5V traces] -> [device's components].

        -Input protection: A TVS diode was placed before the switching buck in order to protect it from high-voltage spikes. After it there are 2 capacitors, the 100uF is needed to avoid voltage drops at a low frequency, the 100 nF is needed to cover high frequency voltage drops.

        -Current budget: To projecting the power supply an estimated current consumption was needed.
        Sensors (from project requirements): 30mA
        Voltage dividers (I = V/R, see section 2.3): 1.5mA
        Comparators (2 × TLV9024-Q1, 8 channels × 35 µA): 0.3mA
        Logic block: 1mA
        LEDs (5 × 2 mA): 10mA
        Relay coil: 31mA
        Total: 75mA

        The board drwas about 75mA, which is 10% of the buck's rated current, leavil a big margin for thermal derating, startup inrush and potential future changes.
        With P_out = 5V x 75mA = 0,38W, with an 85% effincency at 12V(from the datasheet), P_in will be about 0.44W.

        -Regulator choice:
        The switching buck was chosen in order to have a better thermal efficency, the LDO would have had too much power to dissipate and the buck + LDO design was overkilled. (see decision.md)
        The chip was specifically chosen sinche it had a big margin for voltage spike and output current, Wide frequency range(200 kHz-2,2MHz).        


        -Calcolation
        TVS diode: SMBJ18A-E3/5B, standoff tension 18V, right above the 17V max limit, breakdown tension 20V and clamping tension 29,2V, well below the V_in max of the buck(42V).

        C1: Is placed to stop the input from ringing when the board is connected. To do this it must be much larger than c2(x4) and its resistance should be close to the value that most damps c2 and the cable.
        Assumin a meter long cable and conidering the loss of capacity in the cold, the best option was a 100uF 50V electrolitic capacitor, being comfortably above the minimum capacity and having a simular resistance.

        C2: X7R 4.7 µF / 50 V ceramic capacitor placed right next to the buck converter, and it handles the fast current pulses the converter draws while switching(capacity as in the data sheet).

        L: The value(4.7uH) was taken from the datasheet reference design. The peak current was calculate in the worst possible case, startup, when the capacitors on teh 5V rail are fully discharged. The result was 0.40A, the chosen inductor(Coilcraft XFL4020-472MEC) has a saturation current of 2.7A.

        R2 & R3: The resistors values (1M & 187k) were taken from the datasheet design. Using the reference FB voltage (0.778V) the V_OUT will be about 4.94V, about 1.2% lower than the reference 5V, which is an acceptable error.

        C6: The requested minimum capacity was about 10uF, a 22uF x7R ceramic capacitor was chosen. The larger value is a direct consequence that thsi kind of capacitors lose part of the capacity with a DC voltage applied to it. With 22uF the calculated ripple(1.1mV) is well below the 10mV target.

        FB1: A 600 Ω @ 100 MHz ferite bead was placed on the relay branch, so that the coil current step does not affect the rest of the 5V rail. The coil draws about 28mA, well below the 500mA rating, and the voltage drop is only 6mV, the relay still gets 4.99mV.

        -Simulation
        All the pictures of the simulation are in the [/../sim/power] folder.

        -Relay coil supply: To prevent noise generated by the coil from disturbing the rest of the circuit, the relay coil is feed through a different branch of the regulated 5V rail, which is filtered.
        A ferrite bead(Murata BLM18KG601SH1D, 600 Ω at 100 MHz) divides this branch, while 2 capacitors placed near the relay coil complete the filter.
        A 10uF capacitor acts like locacl charge reservoir, to avoid voltage drops in the logic rail at turn on. And a 100nF capacitor redirects high frequency noise to the ground.



    2.2 Inputs
    2.3 Analog
    2.4 Logic
    2.5 Output
    2.6 Connectors
        All the external connections use connectors frotm the same family(Molex Micro-Fit 3.0), as they provide security through a locking latch and polarized architecture. !TODO da finire



