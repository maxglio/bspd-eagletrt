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
    2.2 Inputs
    2.3 Analog
    2.4 Logic
    2.5 Output
    2.6 Connectors
        All the external connections use connectors frotm the same family(Molex Micro-Fit 3.0), as they provide security through a locking latch and polarized architecture. !TODO da finire



