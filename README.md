# Thermal: Output Wires

How hot do a power die's output bond wires run? Seven 1 mil (25.4 µm) gold wires, wedge-bonded from the drain pad on a SiC die
to a copper package lead, carry the device's current. **Everything above the flange is embedded in a mould compound — the
overmold** — so the wires lose their heat into the mould, and through it to the flange, which is held at `Tcase` = 85 °C. The
wires are solved as one-dimensional conductors along their true 3D paths, with gold's electrical and thermal conductivity
following the temperature; the rest of the package is solved in 3D.

The method is taught in the **Thermal** chapter of **Help ▸ circuitRF Documentation**. That chapter's numbers are for the
six-wire version of this workspace that ships with circuitRF; the numbers below are for this one.

## This copy is solved

Clone it with **File ▸ Clone Workspace…** (or `git clone`) and it opens with every result in `results/`: the EM setup's
S-parameters, the `DcSweep`, `RfHarmonics` and `FromHB` thermal runs of `Output` with their meshes and temperature fields, the
`DcSweep` of `Drawn Wires`, and the harmonic-balance sweeps of `Amplifier` and `Amplifier Wires`. The field plots draw at once and
nothing needs to be run first. Each result records a fingerprint of every file it was solved from, so it is shown as current
until you change the design. The EM setup's run folder is not kept; running `EM` rebuilds it.

**What to expect when you press Run** (an Apple M4, the Release build): `EM` **1 min 27 s** in Palace and about **6 GB**;
`DcSweep` about **7 s**, `RfHarmonics` about **7 s**, `FromHB` about **12 s** once the EM result exists; `Drawn Wires` `DcSweep`
about **8 s**; the `Amplifier Wires` sweep under a second. Thermal needs **Gmsh**, and the EM setup **Palace**; the first run
without either offers to install it.

## What is here

| Cell | View | What it is |
|---|---|---|
| **Pads** | layout, µm | The die: 100 µm of SiC on a 25 µm gold-tin die attach, with a 10 µm gold drain pad along its edge, and the lead finger (10 µm gold) — plus **`Pads.wBond`**, the seven wires |
| **Die** | layout, µm | The same artwork without the wires, for *Drawn Wires* |
| **Output** | 3D view, µm | The die placed as `U1` on a copper–molybdenum flange; a copper lead on an alumina standoff under the finger; the mould over all of it; two ports, one probe per wire; the setups `EM`, `DcSweep`, `RfHarmonics` and `FromHB` |
| **Drawn Wires** | 3D view, µm | The same model with the seven wires drawn in the 3D view instead of read from `Pads.wBond`; the setup `DcSweep` |
| **Amplifier** | schematic (test bench) | `Amplifier.csch`: the harmonic-balance drive sweep `FromHB` takes its currents from |
| **Amplifier Wires** | schematic (test bench) | The same drive with a wBond carrying `Pads.wBond`'s wires in place of the EM block: the wire temperature with no 3D run |

**The wires.** Seven identical wires, 127 µm apart, at y = −375, −248, −121, 6, 133, 260 and 387 µm. Each leaves the drain pad at
x = 523 µm, rises steeply to its highest point, 274 µm above the pad, at x = 703 µm, and falls in a long run to the lead at
x = 1,250 µm: a 727 µm span, about 0.97 mm of wire, with a 50 µm foot at each end. The wires are numbered `w1` to `w7` across
the row, so `w1` and `w7` are the edge wires and `w4` the centre one.

Every material is one of circuitRF's generic materials, from `tech/generic-materials.cmat`. The mould's glass transition is taken
as 150 °C, and every wire probe's *Limit* is set to it, so a run says when and where a wire first passes it.

## How it is modelled

- **The overmold.** The box `mould` (generic mould compound, 0.9 W/(m·K)) fills everything from the flange up to 600 µm. The
  wires run through it, and each wire element loses heat into the mould around it.
- **The wires** are the seven wires of `Pads.wBond`, reached through the layout. Each is a chain of line elements along its 3D
  path (feet included), its section πd²/4, its feet bonded perfectly to their pads — the pad's surface under a foot is one
  temperature and one potential with the wire's heel.
- **The pad and the lead are one potential each.** Their resistance is below a hundredth of a milliohm, against about 47 mΩ for a
  wire at 20 °C, so the run solves no potential inside them and says so in its notes (*Balance ▸ EquipotentialBelow*).
- **The flange's bottom** is held at `Tcase`. Every other outer face is insulated.

## DcSweep: a DC current

Port 1's current enters through the lead finger's outer edge and leaves through the drain pad's inner edge; `Idc` sweeps 6 → 18 A
for the seven wires together.

| Idc | Hottest wire (`w4`) | Edge wire (`w1`) |
|---|---|---|
| 6 A | 110.9 °C | 108.4 °C |
| 10 A | 165.1 °C | 157.8 °C |
| 14 A | 279.4 °C | 262.9 °C |
| 18 A | 595.8 °C | 556.5 °C |

At DC the seven wires share the current almost equally — at 14 A the edge wires carry 2.03 A and the centre one 1.98 A — and the
centre ones run hottest anyway, because the mould between their neighbours is warmer. Every wire passes the 150 °C limit by 10 A.

**No runaway inside this sweep.** Gold's resistance rises with temperature and the heat rises with it; past some current nothing
balances, and a run that reaches it brackets it and says so. Seven wires at 18 A carry 2.6 A each, and every point here has a
steady state. Raise the sweep's top to find where it ends.

## RfHarmonics: RF current at 2 GHz

`Idc` = 4 A with a fundamental `I1` at 2 GHz stated as a **Peak** current, swept 0 → 12 A, and a second harmonic of 1 A stated as
**RMS**. At RF the current does not share equally: it divides by the wires' inductance, and **the two edge wires carry the
most**. At 8 A peak the edge wires reach **228.7 °C** and the centre one **165.3 °C**; at 12 A, **454.8 °C** against
**270.5 °C**. `w1` passes the limit by 8 A.

## FromHB: the currents of a power amplifier

The **Amplifier** test bench is a generic GaN-like FET (circuitRF's Angelov model) biased in class AB, its drain on this view's EM
result (the `SnP` block `XOUT`), fed from 40 V through a choke on the lead side, into a 5 Ω load. **It exists to drive currents
through the wires, not to be a good amplifier:** nothing is matched. Its harmonic-balance sweep takes `Pin` from 10 to 28 dBm, and
at every drive level the thermal run takes each port's DC and harmonic currents (pin p is port p). Both ports are referenced to
the flange, across the die and the standoff, so the run solves them as a balanced pair.

**Run `EM` first** when you change the design. `FromHB` reads the S-parameters `EM` writes; without them it refuses and says so.
The EM result puts the wires' series inductance at 2 GHz at **219.4 pH**.

- At `Pin` = 28 dBm the circuit gives **49.97 dBm** from **4.65 A**, and the edge wires reach **190.7 °C**; the centre one stays
  just under the limit, at **149.1 °C**.
- `w1` passes 150 °C between `Pin` = 14 dBm (147.8 °C) and 16 dBm (164.0 °C); the run's notes give the interpolated drive.

One way only: the circuit saw the wires at the temperature their S-parameters were solved at; the thermal run reports what they
actually reach. The DC resistance of the EM block is its lowest frequency's (0.1 GHz), so the drain sits a few volts below the
lead at DC; the DC **current** through the wires, which is what heats them, is the circuit's own.

## Amplifier Wires: the quick check, with no 3D run

The **Amplifier Wires** test bench is the `Amplifier` bench's drive — the same FET, bias, choke, load and `Pin` sweep 10 → 28 dBm
— with a **wBond component carrying the seven wires of `Pads.wBond`** where `Amplifier` has the EM block. Its temperature is
**solved** (the checkbox on its `Temp` row is clear), between 125 °C at the die end and 85 °C at the lead end, and every point of
the sweep reports **`WireTemp`**: the hottest wire, at its hottest point. It needs no EM run and no Gmsh.

| Pin | WireTemp |
|---|---|
| 10 dBm | 153.2 °C |
| 20 dBm | 364.4 °C |
| 28 dBm | 396.4 °C |

It peaks at **399.0 °C** at 24 dBm. Against `FromHB`'s 190.7 °C at 28 dBm, **the quick check runs hotter because no heat leaves
through a wire's sides**: it has no mould, so every watt flows along the wire to its ends, and it is an upper bound. The circuit is
not quite `Amplifier`'s either — its output network is the wires alone, over the flange as their ground plane, not the EM block of
the whole package — so at 28 dBm it gives **51.44 dBm** from **5.45 A**. The component's `GroundPlane` is on: `Pads.wBond` stores
it off, because the 3D EM setup has its own flange, and a placed wBond needs a return path.

The bench carries **its own copy** of the wires (the wBond's *Source* is *Carried*), so an edit to `Pads.wBond` reaches it only
when the copy is replaced — open the wBond from the bench, or re-place it from `Pads.wBond`.

## Drawn Wires: one wire model, two ways to draw it

The same DcSweep with the seven wires drawn in the 3D view (one wire and an array of seven, 127 µm apart) instead of read from
`Pads.wBond`: every wire's hottest point is the same in both, to every printed digit — `w4` 595.8 °C and `w1` 556.5 °C at
18 A. The two are one thermal problem.

## See it

`Output` carries four field plots — a section along wire 4 at 14 A DC (drawn when you open the results), across the seven wires
at DC and at RF, and in plan through the loops at RF. Tick one in the tree to draw it. Each wire is coloured from its own solved
temperature along its length, over the mould around it. Right-click the lead's top face ▸ **Plot Temperature** for its surface.

## The settings chosen for speed

**The EM setup at the Draft preset.** *Standard* reads the wires' inductance a few percent higher and takes several times longer;
Draft is the setting for a run you repeat, Standard for the number you quote. **The pad and the lead as one potential each**:
with *Balance ▸ EquipotentialBelow* = 0 every conductor's potential is solved in 3D, at about twice the time. **Element order 1**
and one element through thin solids (`MinThroughThickness` 1): the 10 µm gold pads set the element size where they are, and
order 2 multiplies the unknowns by about eight. None of the three was re-measured for this seven-wire design.

## Things to try

- Change `Tcase`, or the sweep on a setup's page — raise `DcSweep`'s top current until it reports the runaway.
- Move the wires closer together in `Pads.wBond` (the *wBond* window) and run `RfHarmonics`: the edge wires' share grows.
- Switch the EM setup to *Standard* and run `EM`, then `FromHB`, to see what the preset costs here.
