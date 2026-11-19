# SKU-4082 Inventory Replenishment & Disruption Simulation

![40-Week Inventory Sawtooth & Safety Stock Buffer](sawtooth_chart.png)

## Overview
This simple excel model simulates a weekly-reviewed **(s, Q) inventory replenishment policy** for `SKU-4082` across 40 operating weeks, alongside a disruption calculator that evaluates whether to pay for expedited freight when an inbound shipment is delayed.

## Key Results
* **Replenishment Policy:** Order **360 units (15 full pallets)** whenever **Inventory Position** (On-Hand + In-Transit Orders) drops to or below **344 units** (`ROP = 344`, including **67 units of Safety Stock** for a 95% service level).
* **Pallet Optimization:** Rounding theoretical EOQ (`358.43 units`) up to `360 units` (`24 units/pallet`) adds only **$0.06/year** in cycle cost while eliminating partial-pallet breakdown labor at receiving.
* **40-Week Backtest:** Achieved a **100% fill rate** with zero stockouts, absorbing a Week 14 demand spike (`192 units`) down to **43 units** on hand.
* **Expedite Trade-Off (Week 31 Delay):** A 1-week delivery delay exposes the warehouse to **26 lost units** (`$1,170` in lost gross margin at `$45/unit`). Paying a **$450 flat expedite fee** saves **$720 in net profit** (**1.6x net ROI**; breakeven = **10 units**).

## Workbook Structure
* **`Executive Summary`:** Plain-language business brief and 40-week inventory sawtooth chart.
* **`SKU-4082`:** Parameter inputs, EOQ and Safety Stock calculations, and the 40-week simulation ledger built with `INDEX` and `SUMIFS`.
* **`Disruption Analysis`:** Dynamic expedite cost-vs.-margin decision calculator.

## Monte Carlo Stochastic Stress-Test (Python / NumPy)

To stress-test the deterministic Excel policy (`ROP = 344`, `SS = 67`, `Q = 360`) against unseen demand spikes and supplier transit delays, I built a **5,000-trial Monte Carlo simulation** (`200,000` simulated operating weeks per regime) in Python (`SKU_4082_Monte_Carlo_Simulation.ipynb`):

| Simulation Regime (5,000 Trials × 40 Weeks) | Policy (`ROP` / `SS`) | Mean Fill Rate | 40-Week Window Stockout Prob | Mean Lost Sales (Units) |
| :--- | :---: | :---: | :---: | :---: |
| **Scenario A:** Demand Volatility Only ($L = 2\text{ wks}$) | `344` / `67` | **99.97%** | **10.2%** | **1.7** |
| **Scenario B:** Demand + 15% Carrier Delay ($L \in \{2, 3\}$) | `344` / `67` | **98.74%** | **73.2%** | **70.1** |
| **Mitigated Policy:** Combined Volatility Buffer ($\bar{L}, \sigma_L$) | `405` / `107` | **99.61%** | **42.2%** | **21.7** |

### Key Analytical Findings
* **Lead-Time Volatility Exposure:** Because standard continuous-review safety stock ($SS = z \cdot \sigma_D \sqrt{L} = 67$) assumes deterministic lead times ($\sigma_L = 0$), introducing a 15% probability of a 1-week carrier delay increases 40-week stockout window risk from **10.2% to 73.2%** and inflates expected lost sales from **1.7 to 70.1 units** (**$3,155** gross margin at risk).
* **Combined Buffer Optimization:** Recalibrating safety stock via King's dual-volatility equation ($SS = z \sqrt{\bar{L}\sigma_D^2 + \bar{D}^2\sigma_L^2} = 107$, raising `ROP` to `405`) cuts expected lost sales by **69%** (`70.1` $\rightarrow$ `21.7` units), protecting **$2,178** in gross margin for **~$15.60** in incremental annual carrying cost.
