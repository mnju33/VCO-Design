# VCO Design

Differential MIM capacitor bank design for a broadband, low-jitter VCO targeting a MIPI display interface.

## Scope

This repository documents the capacitor bank design supported by the project research notes: device selection, bank configuration, PCell layout, pins, routing, grounding, and EMX verification. Completion of the entire VCO or its final measured performance is outside the documented scope.

## Bank Configuration and Device Selection

- Started from extraction results for 1,088 unit MIM capacitor geometries.
- Ranked candidates by Q at 9 GHz and skipped candidates within 0.5 fF of an already selected capacitance, leaving 199 geometries.
- Implemented banks with 15, 31, or 63 capacitors per side: 30, 62, or 126 capacitors across both differential sides.
- Generated 597 bank variants: 199 geometries × 3 bank sizes.
- Used oscp/oscm common terminals and capp/capm individual taps.

## PCell Layout

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

## Layout Revision and Verification

The initial layouts contained label-placement errors and lacked actual pin objects. They were rebuilt using MIM PCells and explicit pins to match the intended circuit implementation.

- Checked all 597 base layout views, including 43,382 PCell instances and spacing/pin order across 21,691 rows.
- Verified the order of 86,764 M8/pin shapes.
- Recorded 18–36 M9–M8 via cuts per contact.
- Inspected both layout and layout_mim_v2 views.
- The final pin-resizing entry reports 1,193 of 1,194 views updated and verified. The remaining locked view requires a final status check.

## EMX Verification Status

Initial extraction indicated that differences in P/M port placement and ground contacts affected Q comparisons. Capacitance and Q values from the layout before pin correction remain reference data; they do not establish final performance of the corrected layout.

The remaining verification steps are:

1. Confirm that the latest verified GDS and port definitions match.
2. Compare the label-centered 1 × 1 µm injection port with a port definition that includes pin attributes on a representative cell.
3. Check convergence of 9 GHz capacitance and Q across mesh settings.
4. Compare P/M symmetry and position-dependent resistance.
5. Re-extract all 597 banks after fixing the extraction conditions, then update the design calculator.

## Source Material

[VCO weekly research notes — W11](https://docs.google.com/spreadsheets/d/10oYxRaVc69ShrGxXwJi_bWyr5pR38Fh5EPvvSAdEf2o/edit#gid=340642055): capacitor bank design, layout reconstruction, and verification status.

Original PDK files and circuit databases are not included.

## Related Project

[QLL Design](https://github.com/mnju33/QLL-Design) — injection-locked quadrature clock circuit design and simulation.
