# VCO Design

Varactor, inductor, and differential MIM capacitor bank design for a broadband, low-jitter VCO targeting a MIPI display interface.

## Scope

This repository covers the VCO tank design: MOS varactor design and layout, inductor design, and differential MIM capacitor bank implementation. The W07–W11 notes document the complementary cross-coupled LC-VCO architecture, inductor EMX comparisons, tank calculations, varactor simulations, and capacitor bank layout checks described below. The results do not establish final silicon performance of the complete VCO.

## VCO Architecture

W07 records the circuit configuration of a complementary cross-coupled LC-VCO targeting 8–10 GHz. Subsequent work sized the resonant tank, evaluated startup transconductance and tuning gain, and developed the passive components.

## Tank Sizing and Tuning Budget

The W09 calculator used L = 200 pH to size an 8–10 GHz LC tank:

| Parameter | W09 calculation |
| --- | --- |
| Required tank capacitance at 8 / 9 / 10 GHz | 1978.9 / 1563.6 / 1266.5 fF |
| Required capacitance span | 712.4 fF |
| Selected varactor sizing | B = 8, G = 6 |
| Varactor capacitance span | Approximately 42.3 fF |
| Bank configuration | 31 identical units, 5-bit control |
| Target capacitance step per bank unit | 21.617 fF |
| Preliminary MIM size | 4 × 8 µm |
| Estimated overlap ratio | 1.95 |
| Estimated average KVCO | +202.9 MHz/V |
| Fixed capacitance budget | 562.3 fF |
| Calculated frequency range | Approximately 7.995–10 GHz |

The calculation used C_tank = C_fixed + C_par + C_var + C_bank(k), with C_bank(k) = 31 × Coff + k × (Con − Coff). The parasitic capacitance of 150 fF was an assumption. MIM on/off values for the 4 × 8 µm cell were approximate graph readings, and the overlap margin does not guarantee PVT coverage. These are preliminary sizing values to be updated with extracted routing parasitics and the final component selection.

## MOS Varactor Design and Layout

### Device Sizing

W09 compared B/G combinations (10, 3), (6, 6), and (8, 6) at 9 GHz over Vtune = 0.3–0.9 V.

| B / G | Cmax | Cmin | Q at Cmax |
| --- | --- | --- | --- |
| 10 / 3 | 59.69 fF | 32.71 fF | 28.28 |
| 6 / 6 | 71.19 fF | 38.90 fF | 26.67 |
| 8 / 6 | 94.30 fF | 52.00 fF | 25.81 |

Capacitance scaled approximately with B × G, while Cmax/Cmin remained approximately 1.82. The (8, 6) configuration was selected for the preliminary tuning budget to provide greater adjacent-code overlap.

### Layout Automation and Extraction

- Automated layout and routing from the existing moscap_rf OASIS structure.
- Reference cell: var_oasis_auto_001, using two devices with W = L = 400 nm, eight fingers per group, and six groups.
- Defined cp, cn, vtune, and gnda pins.
- Ran Calibre LVS and xRC PEX including resistance, capacitance, and coupling capacitance.
- Compared the schematic against rcc_v0; generated rcc_v1 after removing side PO, PP, and M1 structures.
- Swept five values each for W, L, B, and G: 625 combinations, simulated at 9 GHz across FF/TT/SS corners, Vtune = 0–1.2 V in 0.1 V steps, at 27 °C.

### W10 Verification Results

| Reference-cell metric | Schematic | rcc_v0 PEX |
| --- | --- | --- |
| Capacitance, Vtune 0 → 1.2 V, TT | 74.33 → 36.18 fF | 82.67 → 44.59 fF |
| Capacitance span | 38.15 fF | 38.08 fF |
| Cmax/Cmin | 2.05 | 1.85 |
| Q at Vtune = 0.6 V | 38.96 | 30.47 |

Extraction added approximately 8.32–8.41 fF across the tested corners and voltages, while Q decreased by 14.6–25.0%. The nearly unchanged tuning span and reduced Q indicate added parasitic capacitance and routing loss.

All 625 combinations passed LVS, and the 24,375-point dataset had no missing or duplicate samples. One geometry produced 27 negative-C/Q samples; the notes identify a filtered 624-geometry, 24,336-row dataset for subsequent optimization work. The highest minimum-Q geometry across corners had only 4.16–5.34 fF of capacitance, so Q ranking alone was insufficient to satisfy the tuning budget.

Nineteen via-spacing issues were corrected. The unit-cell check with FULL_CHIP disabled retained nine density violations; full-chip boundary, I/O, and ESD checks remained unresolved. This is not a claim of complete DRC closure.

## Inductor Design

### Structure Selection

W07 selected a differential symmetric octagonal inductor with a center tap as the baseline design direction. Starting guidelines included an inner diameter of at least five times the conductor width and spacing around three times the minimum allowed spacing. These were initial design guidelines, not documented final geometry dimensions.

The design review considered conductor width, outer/inner diameter, turn count, substrate coupling, self-resonance, and the trade-off between inductance, loss, and tuning range. Ground shielding, twisted four-port structures, and tail-inductor filtering were literature-based options rather than implemented results in these notes.

### Metal-Stack EMX Comparison

W08 compared seven metal configurations using the same inductor geometry over a recorded 0–100 GHz sweep:

| Variant | Metal configuration |
| --- | --- |
| 0 | M9 |
| 1 | M9 + M8 |
| 2 | M9 + M8 + M7 |
| 3 | M9 + M7 |
| 4 | M9 + M8 + M7 + M6 |
| 5 | M9 + M6 |
| 6 | M9 + M6 with fewer vias |

The extracted inductance, Q, and series-resistance curves nearly overlapped across all seven variants.

| Metric | Approximate W08 EMX result |
| --- | --- |
| Low-frequency inductance | 170–180 pH |
| Q near 10 GHz | 8–9 |
| Peak Q | Approximately 19 at 45–50 GHz |
| Configuration-dependent differences | Small differences mainly above 60 GHz |

Adding lower metal layers or reducing the via count produced little change in the target 8–10 GHz band for this geometry. The notes proposed thick top metal, via resistance, and added substrate capacitance as possible explanations; these were hypotheses requiring further checks.

### Design Implications and Remaining Work

The W08 ideal calculator assumed Q = 30 while comparing L = 100/150 pH, 3–6-bit thermometer-coded banks, and unit/varactor capacitance steps of 30/40/50 fF. Those 72 sizing cases were preliminary calculations, separate from the EMX results. The selected 150 pH / 5-bit / 40 fF case had an adjacent-code overlap ratio of 1, leaving no overlap margin.

Because the extracted Q near 10 GHz was approximately 8–9, the ideal calculator's startup-current and power estimates require recalculation using extracted loss. W09 subsequently used L = 200 pH for a revised tuning budget; that value is a calculator input, not the measured or extracted inductance of the W08 geometry.

The next recorded steps were to vary turn count, conductor width, and inner diameter; check PDK metal thickness and sheet resistance; and incorporate extracted L and Q into tank sizing. Final geometry dimensions and self-resonant frequency are not specified in the readable entries.

## MIM Capacitor Characterization

W10 generated 1,681 MIM layouts and ran LVS/PEX. Spectre failed for 593 small geometries because of a division-by-zero issue in the PDK model's resistance calculation, leaving 1,088 valid simulation geometries.

- Compared schematic and PEX across FF/TT/SS at 27 °C: extracted C increased by 2.19–7.97%, while Q changed by −5.68% to +6.66%.
- Checked EMX mesh convergence before extracting all 1,088 valid capacitors over 0.1–50 GHz in 0.1 GHz steps, with e/t/v = 0.1/0.1/0.1 µm.
- At 9 GHz, the W = 5.8 µm, L = 6.0 / 6.2 / 6.4 µm geometries yielded approximately 75.58 / 77.96 / 80.37 fF and Q = 479.5 / 478.2 / 477.1.
- Interpreted the W = 5.8 µm family as a favorable trend rather than treating the small Q differences as a definitive ranking, because Q was sensitive to the edge mesh.
- Used the EMX unit-capacitor results as the basis for the subsequent bank layouts.

## Bank Configuration and Device Selection

- Started from extraction results for 1,088 unit MIM capacitor geometries.
- Ranked candidates by Q at 9 GHz and skipped candidates within 0.5 fF of an already selected capacitance, leaving 199 geometries.
- Implemented banks with 15, 31, or 63 capacitors per side: 30, 62, or 126 capacitors across both differential sides.
- Generated 597 bank variants: 199 geometries × 3 bank sizes.
- Used oscp/oscm common terminals and capp/capm individual taps.

## Capacitor Bank PCell Layout

| Item | Design |
| --- | --- |
| Device | tsmcN65/mimcap_um_rf PCell |
| Placement | One capacitor on each side of every row |
| Central M8 spacing between left and right capacitors | 1.50 µm |
| Outer M9 row spacing | 0.84 µm |
| Pin order per row | oscp[i] → capp[i] → capm[i] → oscm[i] |
| Pin access directions | oscp: left; capp: right; capm: left; oscm: right |
| Oscillator pins | M8/pin sized to match M9 |
| Capacitor pins | Located on the protruding M8 sections |
| Vias | Geometry-dependent sizing and M9–M8 cut count |
| Ground | One vssa M5 contact per side; all BULK terminals connected through a common M5 network |

## Capacitor Bank Layout Revision and Verification

The initial layouts contained label-placement errors and lacked actual pin objects. They were rebuilt using MIM PCells and explicit pins to match the intended circuit implementation.

- Checked all 597 base layout views, including 43,382 PCell instances and spacing/pin order across 21,691 rows.
- Verified the order of 86,764 M8/pin shapes.
- Recorded 18–36 M9–M8 via cuts per contact.
- Inspected both layout and layout_mim_v2 views.
- The final pin-resizing entry reports 1,193 of 1,194 views updated and verified. The remaining locked view requires a final status check.

## Capacitor Bank EMX Verification Status

Initial extraction indicated that differences in P/M port placement and ground contacts affected Q comparisons. Capacitance and Q values from the layout before pin correction remain reference data; they do not establish final performance of the corrected layout.

The remaining verification steps are:

1. Confirm that the latest verified GDS and port definitions match.
2. Compare the label-centered 1 × 1 µm injection port with a port definition that includes pin attributes on a representative cell.
3. Check convergence of 9 GHz capacitance and Q across mesh settings.
4. Compare P/M symmetry and position-dependent resistance.
5. Re-extract all 597 banks after fixing the extraction conditions, then update the design calculator.

## Source Material

- [W07 — LC-VCO architecture and inductor structure selection](https://docs.google.com/spreadsheets/d/10oYxRaVc69ShrGxXwJi_bWyr5pR38Fh5EPvvSAdEf2o/edit#gid=368872168).
- [W08 — inductor metal-stack EMX comparison and preliminary sizing](https://docs.google.com/spreadsheets/d/10oYxRaVc69ShrGxXwJi_bWyr5pR38Fh5EPvvSAdEf2o/edit#gid=452662224).
- [W09 — varactor sizing and LC tank calculations](https://docs.google.com/spreadsheets/d/10oYxRaVc69ShrGxXwJi_bWyr5pR38Fh5EPvvSAdEf2o/edit#gid=2063818175).
- [W10 — varactor layout/LVS/PEX and MIM characterization](https://docs.google.com/spreadsheets/d/10oYxRaVc69ShrGxXwJi_bWyr5pR38Fh5EPvvSAdEf2o/edit#gid=1811760778).

[VCO weekly research notes — W11](https://docs.google.com/spreadsheets/d/10oYxRaVc69ShrGxXwJi_bWyr5pR38Fh5EPvvSAdEf2o/edit#gid=340642055): capacitor bank design, layout reconstruction, and verification status.

Original PDK files and circuit databases are not included.

## Related Project

[QLL Design](https://github.com/mnju33/QLL-Design) — injection-locked quadrature clock circuit design and simulation.
