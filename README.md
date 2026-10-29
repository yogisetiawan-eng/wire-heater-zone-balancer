# Wire Heater Zone Balancer

**Live app:** https://yogisetiawan-eng.github.io/wire-heater-zone-balancer/

A single-page tool for balancing a silicone wire heater on a frame or flange. You enter each side's size, its wire Ω/m and three probe temperatures per side from a test run. The tool keeps one wire zone per side, grades the wire pitch along each side, and sets the wire lengths so that every section reaches the same temperature. A heat map shows measured and predicted temperatures on every surface, with the wire layout drawn at its real pitch.

It runs entirely in the browser. There is no install, no server and no data upload.

## Workflow

1. **Design for the target.** Enter the target temperature, operating ambient and wire Ω/m. The heater power is calculated from the target. Each section's surface loss at the target is natural convection on a vertical surface (Churchill–Chu) plus radiation at your emissivity, times an extra loss factor for mounts and attached panels. The total wire then follows from the power and supply: in series `R = V²/P`, `L = R / Ω/m`. That wire is shared between the zones in proportion to each zone's heat loss, so every zone runs at the target. You can choose to spread it evenly by area instead.
2. **Measure actual.** Build that layout, run it at full power and enter T1–T3 per side. The design locks as built at the first reading, so later changes to the target or losses don't move the test layout. The estimate is shown under each reading, and the step card reports the extra loss factor that would make the estimate match. Use that factor for your next design.
3. **Adjusted layout.** Each section's real heat loss is calibrated from the readings, including heat shared through the plate. The wire per section is then recalculated to hold the target. If the readings match the estimate, the adjusted layout is the same as the design.

Heater power is not an input: power and target can't both be fixed, so the app always sizes the power for the target. The first test then shows how far the real frame differs, and step 3 corrects every zone.

**Fill readings with estimate** shows what the adjusted step would do with perfect readings. **Clear readings** empties the probe inputs for a new test.

## Inputs

- **Test run:** supply voltage, room temperature, wiring (one series string or one string per side), and wire distribution (even by area, or the wire length you laid in each section).
- **Per side,** listed clockwise from the top:
  - length (mm) and width (mm);
  - wire resistance (Ω/m);
  - test wire for each of its three sections (m);
  - probe temperatures **T1, T2, T3** at 1/6, 1/2 and 5/6 of the side's length, measured clockwise. Top runs left → right, right runs top → bottom, bottom runs right → left, and left runs bottom → top.
- **Plate:** thickness and conductivity of the front plate and of any backing plate, and whether the sides join at the corners. These set how much heat the sections share through the plate.
- **New layout:** supply voltage, operating ambient, wiring, pitch along each side (graded over the three sections, or uniform), and total power. Total power can be the same as the test, a custom wattage, or whatever is needed to hit a target temperature.

## Outputs

- **Per side:** wire total, change from the test, power and wire load in W/m.
- **Per section:** new power, wire length, pitch, predicted surface temperature and an estimated wire temperature.
- **Heat maps:** measured and predicted, on one shared colour scale with the spec band marked.
- **Use this layout for the next test:** copies the new wire per section into the test inputs, so for the next round you only enter the new probe readings.
- **Copy results for Sheets:** copies the table as tab-separated text. If your sheet uses comma decimals, set its locale to US or UK first.

## Workspaces

- **Save / Load / Delete:** keeps named workspaces in your browser, for example one per test round.
- **Export file / Import file:** saves a workspace as a `.json` file and loads it back. Use this for backups, to move to another computer, or to share with a colleague. Files from the earlier single-probe version import too; width is assumed to be 60 mm, so check it.
- Your current inputs are restored automatically when you reopen the page.

## Method

1. **Sections.** Each side is split into three equal sections, one per probe. Test power per section follows from the wiring:
   - Series: `I = V/ΣRᵢ`, `Pᵢ = I²·Ω/m·Lᵢ`.
   - One string per side: `I_side = V/(Ω/m·L_side)`.
2. **Heat sharing.** The link between neighbouring sections is `Kᵢⱼ = Σ(k·t)·width/spacing`, with corner links when the frame is closed. Each section's own loss to the room is `Gᵢ = (Pᵢ − Σ Kᵢⱼ(Tᵢ − Tⱼ)) / (Tᵢ − T_room)`.
3. **Loss model.** `lossᵢ(ΔT) = Gᵢ·ΔTᵢ,test^(−n)·ΔT^(1+n)`. The default n = 0.12 accounts for convection and radiation rising faster than ΔT.
4. **Graded pitch.** At a uniform temperature no heat flows between sections, so each section gets exactly its own loss: `Pᵢ = Gᵢ·ΔTᵢ,test^(−n)·ΔT^(1+n)`.
5. **Uniform pitch.** Each side's power is spread evenly over its three sections. The side powers are chosen by area-weighted least squares to make the temperature as flat as possible.
6. **Wire.**
   - Series: `R_total = V²/P`, `Lᵢ = R_total·(Pᵢ/P)/Ω/m`.
   - One string per side: `L_side = V²/(P_side·Ω/m)`, split across the sections in proportion to their power.
   - Pitch = section area ÷ wire length. Wire estimate = predicted temperature + W/m × R′.
7. **Prediction.** Predicted temperatures come from solving the full heat balance, sharing included.

An independent implementation (Python/numpy, with a Newton solve and an SLSQP optimum) was compared with the app on 27 cases. These covered series and per-side wiring, graded and uniform pitch, all three power modes, mixed Ω/m, manual section lengths, closed and open frames, stainless with and without aluminium backing, and 2, 3 and 4 sides. Graded-pitch results agree within 1e‑15. Uniform pitch with n > 0 agrees within about 0.05 %.

## Assumptions and limits

- **Use full-on heater power in the test.** If a controller was cycling, the real power was lower than calculated.
- **Each section is represented by its probe.** Graded pitch flattens the probe points, but temperature still varies within a section, and the corner squares are not modelled.
- **Probe noise feeds into the result.** A 1 K probe error shifts that section's heat loss by a few percent. A warning appears if the readings can't be explained by the plate conduction you entered.
- **Losses are referred to room air.** Conduction into mounts, the door or a warm chamber is lumped into each section's heat loss.
- **Wire resistance and supply voltage are treated as constant.** Lead and connector voltage drop is ignored.
- **Pitch and wire temperature are estimates.** Check the insulation rating of your wire with the supplier.
