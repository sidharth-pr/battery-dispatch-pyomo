# Battery dispatch with cycle aging cost (Pyomo)

A small linear program that schedules a 10 MWh / 5 MW grid battery over 24
hours of day-ahead prices, and asks one question: **what changes when
battery degradation is priced instead of ignored?**

## Why

I previously worked with a detailed OpenModelica model of a high-power DC
charging station, generating realistic charging curves from EV battery
ratings and charger power limits. The lesson that stuck with me was
economic: operating behaviour drives the value of energy infrastructure,
and that value is decided at the system level, not inside the component.

Reading the open-source [LEGO model](https://github.com/IEE-TUGraz/LEGO)
(Wogrin et al., *SoftwareX* 2022) from TU Graz's Institute of Electricity
Economics and Energy Innovation, I found the same idea formalized in its
Cycle Depth Stress Function (CDSF): battery cycling is not forbidden, it
is *priced*, so the optimizer trades degradation against market value.
This repository is my "hello world" version of that concept, and my first
project in Pyomo after working on optimization problems in Matlab/Simulink
during my Master's degree.

## The model

- Decision variables: hourly charge, discharge, state of charge
- SoC balance with charge/discharge efficiencies; end-of-day SoC must
  return to the start value, so profit is honest arbitrage
- Objective: arbitrage profit minus a linearized cycle aging cost of
  `replacement_cost / (2 * cycles_to_EOL * capacity)` per MWh discharged

LEGO's CDSF refines this by segmenting SoC into cycle-depth ranges and
pricing deep cycles superlinearly via a power-law stress function. The
linearized version here keeps the model an LP and the code readable.

## Result

| Scenario             | Profit (EUR/day) | Energy discharged (MWh) |
|----------------------|------------------|-------------------------|
| Ignoring degradation | 818              | 23.8                    |
| Pricing degradation  | 431              | 14.5                    |

Pricing aging at ~20 EUR/MWh makes the battery skip the shallow-margin
trades: it cycles roughly 40% less and keeps only the price spreads that
actually pay for the lifetime they consume. The naive schedule's higher
"profit" is an illusion that ignores the cost of wearing out the asset.

![dispatch](dispatch.png)

## Run it

```bash
pip install pyomo highspy matplotlib
python battery_dispatch.py
```

## References

- S. Wogrin, D. A. Tejada-Arango, R. Gaugl, T. Klatzer, U. Bachhiesl,
  "LEGO: The open-source Low-carbon Expansion Generation Optimization
  model," SoftwareX, vol. 19, 2022.
