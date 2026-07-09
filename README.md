# Sallen-Key-Active-Filter-Monte-Carlo-Analysis
# Sensitivity & Yield Analysis of a 2nd-Order Sallen-Key Low-Pass Filter

This repository contains the design, mathematical derivation, and tolerance verification of a active **2nd-Order Sallen-Key Low-Pass Butterworth Filter** using **LTspice**. 

Rather than assuming ideal textbook components, this project introduces a 50-run **Monte Carlo statistical simulation** to analyze how real-world component tolerances (5% resistors, 10% capacitors) affect the filter's cutoff frequency, Q-factor, and passband stability. This level of yield analysis is essential for high-precision analog systems, such as those designed by Texas Instruments.

---

## 1. Design Objectives & Specifications

* **Topology:** Active 2nd-Order Sallen-Key Low-Pass Filter
* **Approximation Type:** Butterworth (maximally flat passband response, $Q = 0.707$)
* **Nominal Cutoff Frequency ($f_c$):** $1\text{ kHz}$
* **Target DC Voltage Gain ($A_v$):** $1.586 \approx 4\text{ dB}$
* **Component Tolerances:** Resistors ($\pm 5\%$), Capacitors ($\pm 10\%$)

---

## 2. Mathematical Derivation

To make the design highly reproducible, the **Equal-Component Design Method** was utilized:

$$R_1 = R_2 = R = 10\text{ k}\Omega$$

To achieve a maximally flat Butterworth response, the filter's Quality Factor ($Q$) must equal $0.707$. The relationship between the DC gain ($A_v$) and $Q$ in this topology is given by:

$$A_v = 3 - \frac{1}{Q} = 3 - \frac{1}{0.707} \approx 1.586$$

The gain is set by the feedback network resistors $R_3$ and $R_4$:

$$A_v = 1 + \frac{R_4}{R_3} = 1.586 \implies \frac{R_4}{R_3} = 0.586$$

By choosing $R_3 = 10\text{ k}\Omega$, we find:

$$R_4 = 5.86\text{ k}\Omega$$

With $R = 10\text{ k}\Omega$ and a target $f_c = 1\text{ kHz}$, the required capacitance ($C$) is calculated as:

$$C = C_1 = C_2 = \frac{1}{2 \pi \cdot f_c \cdot R} = \frac{1}{2 \pi \cdot 1000 \cdot 10000} \approx 15.9\text{ nF}$$

---

## 3. LTspice Simulation Setup

The circuit was constructed in LTspice utilizing a dual power supply configuration ($\pm 15\text{V}$) for the operational amplifier (`UniversalOpAmp2`). To evaluate manufacturing yield, components were assigned statistical uniform distribution formulas:

* **Resistors ($R_1, R_2, R_3$):** `{mc(10k, 0.05)}`
* **Resistor ($R_4$):** `{mc(5.86k, 0.05)}`
* **Capacitors ($C_1, C_2$):** `{mc(15.9n, 0.10)}`

### SPICE Commands Executed:
* `.step param run 1 50 1` (Triggers 50 randomized iterations)
* `.ac dec 100 10 100k` (Performs frequency sweep from 10 Hz to 100 kHz)
<img width="1057" height="610" alt="Schematic" src="https://github.com/user-attachments/assets/8705dd8d-eb88-4bc4-9499-948155743d33" />

---

## 4. Performance & Tolerance Results

Upon running the 50-run simulation, the following deviations from the ideal design were observed:

* **DC Passband Gain:** Remained tightly grouped around the nominal **$4\text{ dB}$** ($1.586\text{ V/V}$).
* **Cutoff Frequency Drift:** Rather than a sharp $1\text{ kHz}$ cutoff, the $-3\text{ dB}$ point (down to $1\text{ dB}$) drifted dynamically between **$910\text{ Hz}$ and $1.09\text{ kHz}$** (an approximate $\pm 9\%$ deviation).
* **Peaking & Q-factor Distortion:** Due to capacitor mismatches, several runs exhibited a minor gain peak ($> 4.2\text{ dB}$) right before the cutoff frequency. This indicates a deviation from the maximally flat Butterworth response into a Chebychev-like response, introducing phase distortion.
<img width="1366" height="671" alt="Monte_Carlo_Plot" src="https://github.com/user-attachments/assets/f8750757-1acf-4a1c-837a-f757c93c4a87" />

---

## 5. Conclusion & Mitigation

For high-volume production, a $\pm 9\%$ cutoff variation is unacceptable in critical signal-conditioning paths (e.g., anti-aliasing filters for delta-sigma ADCs). 

To resolve this, sensitivity analysis indicates that **capacitors are the primary driver of the drift** due to their wider $10\%$ tolerance. Upgrading the capacitors to tighter $2\%$ film capacitors and resistors to $1\%$ thin-film components will successfully compress the cutoff frequency spread to within $\pm 2.5\%$, guaranteeing uniform system performance on the assembly line.
