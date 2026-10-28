# Wire Heater Zone Balancer

A single-page tool for rebalancing a silicone wire heater across several sides of a panel or flange. You enter each side's area, its wire Ω/m and its measured surface temperature from a test run. The tool calculates new wire lengths per side so that every side reaches the same temperature.

It runs entirely in the browser. There is no install, no server and no data upload. Inputs are saved in your own browser.

## Inputs

- **Test run:** supply voltage, room temperature, wiring (one series string or a separate string per side), and wire distribution (even by area with a total length, or the length on each side).
- **Per side:** area (mm²), wire resistance (Ω/m), test wire length (m) and measured surface temperature at steady state (°C).
- **New layout:** supply voltage, operating ambient, wiring, and total power. Total power can be the same as the test, a custom wattage, or whatever is needed to hit a target temperature.

## Outputs per side

New power, new wire length in mm and its change from the test, wire load in W/m, serpentine pitch, predicted surface temperature and an estimated wire temperature. **Copy results for Sheets** copies the table as tab-separated text that pastes into Google Sheets or Excel.

## Method

1. **Test power per side:**
   - Series: `I = V / ΣRᵢ`, `Pᵢ = I²·Ω/mᵢ·Lᵢ`.
   - Separate strings: `Pᵢ = V² / (Ω/mᵢ·Lᵢ)`.
2. **Heat loss per side:** `Gᵢ = Pᵢ / (Tᵢ − T_room)`.
3. **Loss model:** `Pᵢ(ΔT) = Gᵢ·ΔTᵢ,test^(−n)·ΔT^(1+n)`. The default n = 0.12 accounts for convection and radiation rising faster than ΔT. With n = 0 the loss is linear.
4. **Power split:** new power is shared in proportion to `Gᵢ·ΔTᵢ,test^(−n)`, optionally reweighted by `(ΔT_mean/ΔTᵢ)^(k−1)`. With k = 1 every side is predicted to land at the same temperature.
5. **Total power:**
   - Same as test: the summed test power.
   - Custom: the wattage you enter.
   - Target: `Σ Gᵢ·ΔTᵢ,test^(−n) · (T_target − T_ambient)^(1+n)`.
6. **Wire length:**
   - Series: `R_total = V²/P`, `Lᵢ = R_total·(Pᵢ/P) / Ω/mᵢ`.
   - Separate strings: `Lᵢ = V² / (Pᵢ·Ω/mᵢ)`.
7. **Checks:** pitch = area ÷ length, wire estimate = predicted surface temperature + W/m × R′.

The calculation was checked against an independent implementation over series and parallel test and new layouts, all three power modes, mixed Ω/m, n = 0 and n = 0.12, and k = 1.3. All results agreed within 1e‑15.

## Assumptions and limits

- **Sides are treated as independent.** Neighbouring sides share heat at the corners, so one correction usually falls a little short. Rebuild, measure again, switch to *Length per side*, enter the lengths you used and run it again. Two rounds usually bring the spread within about 1 K. A gain k above 1 corrects harder in one step.
- **Use full-on heater power in the test.** If a thermostat or PID was cycling, the real power was lower than calculated. The split between sides still holds, but the target-mode wattage will be overstated.
- **All losses are referred to room air.** Heat paths to a warm chamber, door or gasket shift the target-mode wattage.
- **Wire resistance and supply voltage are treated as constant.** Lead and connector voltage drop is ignored.
- **Pitch and wire temperature are estimates.** Check the insulation rating of your wire with the supplier.

## Hosting on GitHub Pages

1. Put `index.html` at the root of a public repository.
2. Open **Settings → Pages**, then under **Build and deployment** choose *Deploy from a branch*, pick `main` and `/ (root)`, and save.
3. The tool is then live at `https://<username>.github.io/<repository>/`.
