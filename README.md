Here is the complete hardware architecture and wiring breakdown for the NO2 spectrometer using the PS3 KEM-410AAA optical pickup unit (OPU).

Because laser diodes from optical drives will instantly burn out if directly hooked up to a voltage source, we relied on an Op-Amp Constant Current Source (CCS) to drive them safely, and a Transimpedance Amplifier (TIA) to read the micro-currents from the photoreceptor IC (PDIC).



### 1. The Op-Amp Constant Current Laser Driver

To drive both the 650nm (Red DVD) and the 405nm (Blu-ray) lasers, you cannot use constant voltage. The 405nm diode has a high forward voltage (around 4V to 4.4V), while the Red diode is lower (~2.5V).

We used a closed-loop Op-Amp (LMC6484) paired with a PNP transistor and a sense resistor to regulate the exact current.

Driver Wiring:

10K Potentiometer (0V-5V) middle leg -> OP Amp +ve input

Laser GND -> Common GND



      Laser +ve                 OP Amp -ve input & 10K ohm resistor (going to 5V)
             |                        |

transistor Collector     Base      Emitter
                      
                          |
                      1K ohm resistor (going to OP Amp output)
  


### 2. Wiring the KEM-410AAA Lasers

The KEM-410AAA has a complex flexible printed circuit (FPC) ribbon. The laser diodes share a common GND pin in my used model.

 You must identify the common pin using a multimeter and configure two such op amp drivers, one for bluray and one for red dvd laser.

 Lasers can be tuned on/off from microcontroller by first setting the potentiometer to drive lasers optimally, then connecting gnd/power wire of potentiometer to microcontroller digital(0V-5V) output pin to toggle potentiometer output between off and configured state.


### 3. Reading the Photoreceptor IC (PDIC)

To measure the NO2 attenuation, the laser beam is reflected back into the KEM-410AAA's objective lens, hitting the internal photodiode array.

Photodiodes output *current* (in the microamp range), not voltage. To read this with a standard microcontroller ADC, we used the second half of the Op-Amp IC as a **Transimpedance Amplifier (TIA)** to convert that tiny current into a readable voltage.

**PDIC & Receiver Wiring:**

* **PDIC Power:** The photodiode IC on the KEM-410AAA requires its own power to operate its internal amplifiers.
* **VCC:** Connected to a clean 3.3V or 5V rail.
* **GND:** Connected to common ground.


* **Signal Extraction:** The PDIC has multiple outputs (A, B, C, D for the main quadrant, plus E, F for tracking). To get the total light intensity, you tied the main quadrant output pins (A+B+C+D) together to sum their currents.
* **Op-Amp Inverting Input (-):** Connected to the summed PDIC output pins.
* **Op-Amp Non-Inverting Input (+):** Connected to Ground.
* **Feedback Resistor (Rf):** A high-value resistor (e.g., 100kΩ to 1MΩ) connected between the Op-Amp Output and the Inverting Input (-). *This dictates the gain. The higher the resistor, the more sensitive the circuit is to the attenuated light.*
* **Feedback Capacitor (Cf):** A tiny capacitor (e.g., 10pF to 100pF) placed in parallel with the Feedback Resistor to stabilize the op-amp and filter out high-frequency noise.
* **Op-Amp Output:** This is your final analog signal. It outputs a DC voltage directly proportional to the amount of 405nm light hitting the sensor.

WE USED DIFFERNTIAL SPECTROSCOPY TO COMPARE NO2 PRESENCE USING THE RED DVD LASER ALONGSIDE THE BLURAY LASER TO ACT AS BASELINE.

### 4. Microcontroller Integration (The Output)

* The final voltage from the TIA Op-Amp Output was routed into an Analog-to-Digital Converter (ADC) pin on your microcontroller.
* **The Logic:**
1. Turn on the 405nm laser.
2. Read the baseline voltage on the ADC with regular air (V_baseline).
3. Introduce the NO2 gas sample.
4. Read the new voltage (V_sample).
5. Because NO2 absorbs 405nm light, V_sample drops. You calculate the absorbance using the Beer-Lambert Law: A = -\log_{10}(V_{sample} / V_{baseline}).
6. Since NO2 has minimal absorption at 650nm, you could toggle the Red DVD laser to act as a control/calibration beam—proving that any light drop was specifically caused by the 405nm chemical signature of NO2, and not just smoke or condensation in the chamber.
