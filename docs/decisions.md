#22-10-2026 CONNECTORS
For the main connection(Input voltage, sensors and SDC) I decided to use Molex micro-fit 3.0, with 2 pins(43650-0201) for the power input,SDC and with 3 pins(43650-0301) for the sensors.
I chose this kind of connectors mainly for these reasons:
-cheap, about 1 euro each 
-secure lock
-gold plated contacts
-polarized housing
-current rating
-good availability

#23-10-2026 POWER SUPPLY BLOCK
The first and main problem in the power supply is to regulate the unstable voltage (9-17 V) into a regulated 5 V; 3 main options were available:
- Linear regulator (LDO)
- Switching buck
- LDO + SWB

The main issue with the LDO is heat dissipation: in the worst-case scenario, with an estimated load of 75/80 mA and a 12 V voltage drop, we would have to dissipate about 1.6 W.
Since it's not waterproof, the BSPD is supposed to stay in a sealed box, and dissipating 1.6 W in a closed box might become a problem.

The second option, the one I went for, is the switching buck. The main problem with it is the noisy output, but with the right inductor and some capacitors I was able to get a signal at 5 V +- 10 mV.

The third available option was a switching buck followed by an LDO, with the switching buck dropping the voltage (e.g. 5.5 V) and the LDO cleaning the signal, but I thought it wasn't necessary.