# Claus-Process-Simulation

Steady-state simulation of a three-stage Claus sulfur recovery unit built in Aspen Plus V14. The model covers the full process train: reaction furnace, three catalytic converters, three interstage condensers, and three reheaters — from acid gas feed through to tail gas and liquid sulfur product.

---
## Why this matters
H₂S can't be vented. It can't be flared at scale without breaching SO₂ emission limits. And it comes out of every sour gas treating unit in volume — concentrated, continuous, unavoidable. The Claus process converts it into elemental sulfur that can be sold rather than disposed of. That's the industrial reason this process exists.
This simulation runs a three-stage unit on a CO₂-heavy feed: 53.16 mol% CO₂, 36.04 mol% H₂S. That composition makes it harder than the textbook case — furnace temperature control tightens, COS side-reactions become real, and maintaining the 2:1 H₂S:SO₂ ratio at the furnace outlet requires careful air management. The simulation reaches 98.70% recovery and 6,023 kg/hr liquid sulfur — the thermodynamic ceiling for this feed and configuration, not a guaranteed operating result.
The converter temperature profile shows how Claus recovery actually compounds. CR1 rises 47.5°C. CR3 rises 4.3°C. That 4.3°C represents the difference between 97.1% and 98.7% recovery — about 111 kg/hr of additional sulfur, 2.7 t/day. Each stage does less than the last. Three stages is the practical limit before tail gas treatment has to take over.

## Background

Sour gas treating units produce a concentrated acid gas stream — mostly H₂S and CO₂ — that cannot be vented or flared without exceeding SO₂ emission limits. The Claus process converts this H₂S to elemental sulfur, which is a stable, saleable commodity used in fertilizer and sulfuric acid production.

This simulation models a unit receiving acid gas from an amine regeneration column. The feed is CO₂-heavy (53.16 mol% CO₂, 36.04 mol% H₂S), which puts it in the range where furnace temperature management and COS side-reaction handling are non-trivial. Three catalytic stages are simulated, targeting recovery above 98%.

This project is the third in a process simulation portfolio that progresses from distillation (CDU simulation) through gas treating (DEA-based sweetening) to sulfur recovery here. Each project uses the previous one's output as context — the acid gas feed to this Claus unit comes from a sweetening unit of the type modeled in the NGS project.

---

## Process Description

**Thermal stage:** Acid gas, preheated combustion air, and supplementary fuel gas combust in the reaction furnace at sub-stoichiometric air to maintain a 2:1 H₂S:SO₂ ratio in the furnace outlet. The adiabatic equilibrium temperature is 995.6°C at 1.5 bar. Furnace product is cooled to 130°C in the first condenser, recovering the bulk of the sulfur produced.

**Catalytic stages:** Process gas is reheated to 260°C, 220°C, and 190°C before entering each of the three converters. The Claus equilibrium reaction proceeds over alumina catalyst at progressively lower temperatures — lower temperature shifts the equilibrium toward products. Each converter is followed by a condenser at 130°C to remove liquid sulfur and water before the next stage.

**Tail gas:** The OFFGAS stream leaving the final condenser contains residual H₂S, SO₂, N₂, CO₂, H₂O, CO, and H₂. At 98.70% H₂S recovery, the tail gas would require a downstream SCOT or hydrogenation unit to meet regulatory limits above 99.8%.

---

## Simulation Details

| Item | Value |
|---|---|
| Software | Aspen Plus V14 |
| Property package | SR-POLAR (Schwartzentruber-Renon / RK-UNIFAC) |
| Furnace model | RGibbs (Gibbs free energy minimization) |
| Converter model | REquil (specified reaction set, adiabatic) |
| Condenser model | Flash2 (two-phase TP flash) |
| Reheater model | Heater (specified outlet T and P) |
| Components | 16 (CO₂, N₂, H₂S, COS, CS₂, SO₂, SO₃, S₂, S₈, CO, H₂, H₂O, O₂, CH₄, C₂H₆, C₃H₈) |

**Why SR-POLAR:** The system contains H₂S, H₂O, SO₂, and CO₂ — a mix of polar, associating, and acid gas species. Standard cubic EOS packages (PR, SRK) handle this poorly. SR-POLAR incorporates polar parameters per component and uses UNIFAC mixing rules, which gives reliable VLE predictions across the temperature range of the Claus train (130–996°C).

**Reaction set (catalytic converters):**
```
R1:  2H₂S + SO₂ ⇌ 3/8 S₈ + 2H₂O    (primary Claus reaction)
R2:  S₈ ⇌ 4S₂                         (sulfur allotrope speciation)
R3:  COS + H₂O ⇌ CO₂ + H₂S          (COS hydrolysis)
R4:  CS₂ + 2H₂O ⇌ CO₂ + 2H₂S        (CS₂ hydrolysis)
```

---

## Key Results

**Overall sulfur recovery: 98.70%**

| Stage | H₂S at stage inlet (kmol/hr) | Cumulative recovery |
|---|---|---|
| Acid gas feed | 221.99 | — |
| Post-furnace (HOT-GAS) | 54.58 | 75.4% |
| Post-CR1 | 19.83 | 91.1% |
| Post-CR2 | 6.36 | 97.1% |
| Post-CR3 / tail gas | 2.89 | 98.7% |

**Liquid sulfur production:**

| Stream | From | S₂ (kmol/hr) | S₈ (kmol/hr) | Mass (kg/hr) |
|---|---|---|---|---|
| LS | Condenser C | 66.34 | ~0 | 4,316 |
| LS1 | Condenser C1 | 0.22 | 6.84 | 1,775 |
| LS2, LS3 | C2, C3 | — | — | vapor-only (model limit) |

Total liquid sulfur: ~6,023 kg/hr (~144 t/day)

**Converter temperature profile:**

| Unit | Inlet (°C) | Outlet (°C) | ΔT |
|---|---|---|---|
| CR1 | 260 | 307.5 | +47.5°C |
| CR2 | 220 | 236.6 | +16.6°C |
| CR3 | 190 | 194.3 | +4.3°C |

The shrinking ΔT across converters directly reflects diminishing conversion — less reaction, less heat released. CR3's 4.3°C rise accounts for the difference between 97.1% and 98.7% recovery: roughly 111 kg/hr of additional sulfur, or 2.7 t/day.

---

## Feed Conditions

| Parameter | Acid Gas | Air | Fuel Gas |
|---|---|---|---|
| Pressure (bar) | 1.77 | 1.68 | 6.00 |
| Temperature (°C) | 218 | 220 | 40 |
| Flow rate (mol/s) | 171.1 | 181.5 | 3.25 |
| H₂S (mol fr.) | 0.3604 | — | — |
| CO₂ (mol fr.) | 0.5316 | — | 0.0103 |
| H₂O (mol fr.) | 0.0990 | 0.0750 | — |
| O₂ (mol fr.) | — | 0.1950 | — |
| N₂ (mol fr.) | — | 0.7300 | 0.0368 |
| CH₄ (mol fr.) | 0.0090 | — | 0.8946 |

---

## Notes and Limitations

The equilibrium model gives an upper bound on performance. Real Claus units run below equilibrium due to catalyst aging, kinetic limitations at lower temperatures, and heat losses. The 98.70% figure should be read as the thermodynamic ceiling for this feed and configuration, not a guaranteed operating result.

Condensers C2 and C3 return vapor-only results in the simulation. This is a known artifact of the Flash2 model at low residual sulfur concentrations — the solver finds no phase split below a certain threshold. Real plants do recover small amounts of sulfur at these stages. The effect on the overall recovery calculation is minor since the bulk of sulfur is captured at C and C1.

COS and CS₂ from the furnace are both essentially eliminated in the first catalytic bed (98.9% COS reduction across CR1). For feeds with CO₂ content this high, TiO₂ catalyst is preferred over alumina in the first bed due to its higher hydrolysis activity — this was not explicitly modeled but the equilibrium results are consistent with what TiO₂ achieves in practice.

---

## Part of a Simulation Portfolio

This project connects to two earlier simulations in the same portfolio:

- [CDU-Simulation-Optimization](https://github.com/...) — Arabian Light crude distillation, 35-stage column, pump-around optimization, 53.6% furnace duty reduction
- [Natural-Gas-Sweetening](https://github.com/...) — DEA absorption-regeneration loop, >99.9% H₂S removal; the acid gas from that unit motivates the feed specification used here

The next project covers hydrogen production from biogas via dry reforming and water-gas shift
---



## Author

**Shahriar Hossain Suny**  
Department of Chemical Engineering  
Jashore University of Science and Technology (JUST), Bangladesh  
AIChE Member ID: 009905932744  
[LinkedIn](https://www.linkedin.com/in/shahriarhossainsuny/) · [GitHub](https://github.com/shahria-sunny)
