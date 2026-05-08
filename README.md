# mosfet-amplifier
Three-stage CS-CS-CD MOSFET voltage amplifier designed in KiCad and simulated in SPICE
# Three-Stage MOSFET Voltage Amplifier

A discrete CMOS open-loop voltage amplifier designed and simulated in KiCad/SPICE on a 3.3 V supply. Uses a three-stage **CS-CS-CD** topology to combine high voltage gain with output buffering.

> Coursework project for Electronics I (ELE404), Toronto Metropolitan University, Winter 2026.

---

## Topology

**CS-CS-CD** — two common-source gain stages followed by a common-drain (source-follower) output buffer.

- **Stages 1 and 2 (CS):** NMOS amplifying transistors with PMOS active loads; provide the bulk of the voltage gain.
- **Stage 3 (CD):** NMOS source follower biased by a 150 µA current source; buffers the high-impedance internal nodes from the output load.
- **Coupling:** Input and output capacitors block DC; internal stages are direct-coupled so the DC operating point of one stage biases the next.

---

## Performance vs Specifications

All specs met under both unloaded and 10 kΩ-loaded conditions.

| Parameter | Spec | Theoretical | Simulated | Pass |
|---|---|---|---|---|
| Supply voltage | 3.3 V | 3.3 V | 3.3 V | ✅ |
| Input resistance | ≥ 100 kΩ | 195.7 kΩ | 184.35 kΩ | ✅ |
| Open-loop gain (unloaded) | ≥ 60 dB | 80 dB | 65.7 dB | ✅ |
| Open-loop gain (loaded, 10 kΩ) | — | — | 65.1 dB | ✅ |
| Gain reduction under load | ≤ 10% | 9.6% | 6.67% | ✅ |
| Gain per stage | ≤ 40 dB | 40 / 40 / 0 dB | 36 / 31 / −1.9 dB | ✅ |
| −3 dB bandwidth | ≥ 500 kHz | 27 MHz | 1.25 MHz | ✅ |
| Output swing | ≥ 1.5 Vpp | — | 1.53 Vpp | ✅ |
| Total DC power | ≤ 1 mW | 0.86 mW | 0.92 mW | ✅ |
| Max branch current | ≤ 200 µA | 150 µA | 150 µA | ✅ |
| Transistor length | 0.5–5 µm | 0.5 µm | 0.5 µm | ✅ |
| Overdrive voltage | 0.2–0.6 V | 0.25 / 0.30 V | 0.44 / 0.57 V | ✅ |

---

## Design Choices

### Transistor sizing (hand-calculated, then verified in simulation)

Started from `I_D = ½ µ_n C_ox (W/L) V_OV²` with chosen overdrive voltages of V_OVn = 0.25 V and V_OVp = 0.30 V.

| Stage | Device | I_D | W/L |
|---|---|---|---|
| 1 & 2 (CS) | NMOS | 50 µA | 8 µm / 0.5 µm |
| 1 & 2 (CS) | PMOS load | 50 µA | 22.2 µm / 0.5 µm |
| 3 (CD) | NMOS | 150 µA | 16.7 µm / 0.5 µm |

### DC biasing (voltage divider)

Sized R1, R2 to set NMOS gate bias at V_GS1 = V_tn + V_OVn ≈ 0.88 V while keeping R_in ≥ 100 kΩ:

- R1 = 733 kΩ, R2 = 267 kΩ → R_in = R1 ‖ R2 ≈ 195.7 kΩ
- PMOS gate bias set at V_G = V_DD − V_SG = 2.27 V
- Bias-branch divider sized for 10 µA: R3 = 103 kΩ, R4 = 227 kΩ

### Output stage

Stage 3 was deliberately designed for ~0 dB voltage gain. Its purpose is **load isolation** — the source follower's low output impedance (1/g_m3 ≈ 1 kΩ with g_m3 = 1 mA/V) drives the 10 kΩ load while presenting a high-impedance load to Stage 2, so the voltage gain of the earlier stages is preserved.

---

## Theoretical vs Simulated — Honest Discussion

The simulated gain (65.7 dB) was meaningfully lower than the hand-calculated 80 dB. Two main reasons:

1. **Inter-stage loading.** The hand calculation treated each CS stage's gain as `−g_m · (r_on ‖ r_op)` ≈ −100 V/V, which assumes the next stage presents an infinite input impedance. In reality the gate of the next stage and its associated capacitances load the previous stage's output node, reducing per-stage gain from 40 dB to ~36 / 31 dB.
2. **Higher-order MOSFET model effects.** KiCad's transistor model accounts for channel-length modulation more accurately than the simplified square-law equations used by hand, and the simulated overdrive voltages (V_OVn = 0.44 V, V_OVp = 0.57 V) ended up higher than the designed values, shifting the operating point.

The simulated bandwidth (1.25 MHz) was also far below the theoretical 27 MHz. The hand calculation only modeled gate capacitances at one node (C_gs + C_gd contributions ≈ 23.6 fF) and did not capture all parasitics or Miller multiplication of C_gd seen at the high-resistance output nodes of the CS stages. In a high-gain stage, even small capacitances at the output node dominate the pole.

The **gain-bandwidth tradeoff** is the central design tension: increasing g_m (for more gain) requires larger W, which increases parasitic capacitance and reduces bandwidth. Sizing was chosen to comfortably exceed the 500 kHz bandwidth spec while still meeting the 60 dB gain target with margin.

---

## Files

- `MKhan_DesignProject.pdf` — full design report with calculations and simulation plots
- `PR_MOSFET.zip` — KiCad schematic of the three-stage amplifier

## Tools

KiCad (schematic capture), KiCad's SPICE engine (DC operating point, AC sweep, transient analysis)

---

## What I Learned

- Hand calculations are useful as a *starting point*, but multistage amplifier behavior diverges meaningfully from simplified small-signal analysis once inter-stage loading and parasitic capacitances are considered.
- A common-drain output stage gives up voltage gain in exchange for load-driving capability. Without it, loading a 10 kΩ resistor at the output of Stage 2 would have collapsed the gain — the buffer kept gain reduction under 7%.
- The overdrive voltage V_OV is the lever that ties together gain (g_m = 2I_D/V_OV), output swing, and headroom. Choosing V_OV in the 0.2–0.3 V range gave a workable compromise across all three.
