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
        -Requirements: The power supply block is asked to take an unstable current between 9V and 17V and regulate it into a 5V to power the device.

        -Block Overview: [connector] -> [protections] -> [buck] -> [5V traces] -> [device's components].

        -Input protection: A TVS diode was placed before the switching buck in order to protect it from high-voltage spikes. After it there are 2 capacitors, the 100uF is needed to avoid voltage drops at a low frequency, the 100 nF is needed to cover high frequency voltage drops.

        -Current budget: To projecting the power supply an estimated current consumption was needed.
        Sensors (from project requirements): 30mA
        Voltage dividers (I = V/R, see section 2.3): 1.5mA
        Comparators (2 × TLV9024-Q1, 8 channels × 35 µA): 0.3mA
        Logic block: 1mA
        LEDs (5 × 2 mA): 10mA
        Relay coil: 30mA
        Total: 75mA

        The board drwas about 75mA, which is 10% of the buck's rated current, leavil a big margin for thermal derating, startup inrush and potential future changes.
        With P_out = 5V x 75mA = 0,38W, with an 85% effincency at 12V(from the datasheet), P_in will be about 0.44W.

        -Regulator choice: The switching buck was chosen in order to have a better thermal efficency, the LDO would have had too much power to dissipate and the buck + LDO design was overkilled. (see decision.md)
        The chip was specifically chosen sinche it had a big margin for voltage spike and output current, Wide frequency range(200 MHz-2,2GHz).

        -Calcolation

        -Thermal Scenario

        -Simulation(foto e )

        -Relay coil supply: To avoid any issue with 


    2.2 Inputs
    2.3 Analog
    2.4 Logic
    2.5 Output
    2.6 Connectors
        All the external connections use connectors frotm the same family(Molex Micro-Fit 3.0), as they provide security through a locking latch and polarized architecture. !TODO da finire



