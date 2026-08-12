# GTM Explorer

Interactive explorer for the AMIGDALA Policy Lab **Global Trade Model** — bilateral
merchandise trade at HS6 product level, estimated on BACI 2012–2024 and projected to 2070
under paired GDP (EXIOMOD) and CO₂ (TIAM) scenarios.

**Live:** https://ounoughi-chahinez.github.io/gtm-explorer/

## The model

Two stages:

1. **Chapter totals — PPML gravity.** One Poisson regression per HS2 chapter. Combined
   economic size enters as a fixed offset on log(GDP per capita × population) with the
   elasticity held at 1. Estimated regressors: log distance, a damped time trend, and a
   chapter-specific exporter CO₂ elasticity.
2. **Product shares — gradient boosting.** Predicts the change in each HS6 line's logit
   share of its parent chapter, then rescales exactly to the Stage 1 total so adding-up is
   exact by construction.

Two scenario channels sit on top, both **imposed** rather than fitted to trade data:

- **GDP (EXIOMOD).** λ(region, year) = scenario GDP ÷ reference GDP, mapped from 17 EXIOMOD
  regions onto 231 BACI countries. Scales GDP per capita only; λ = 1 through 2024.
- **CO₂ (TIAM).** National emissions per capita, standardised on a scale common to all
  pathways. Coefficients are fitted once on a fixed reference pathway; CO₂ varies only in
  projection, since the TIAM scenarios already diverge inside the estimation window.

## Layers

| Layer | GDP overlay (EXIOMOD) | CO₂ channel (TIAM) |
|---|---|---|
| Scenario *n* | Scenario *n* | Scenario *n* |
| Base | off | off |
| GDP only | Scenario 1 | off |
| CO₂ only | off | Scenario 1 |

Scenarios: 1, 6, 6 low bio, 8, 10 (reference), 15.

## What the model supports

**Estimated** — gravity structure, chapter CO₂ elasticities, HS6 share allocation.
**Imposed** — the unit income elasticity, the TIAM pathway, the EXIOMOD overlay.
**Not supported by the data** — a green reallocation mechanism. The cross-chapter test of
whether fossil-exposed chapters contract faster as exporters decarbonise returns the wrong
sign and is not statistically significant, and the HS6 CO₂ interactions carry negligible
feature importance. Fuels results are a scenario assumption, not a model finding.

**Known artefact** — the CO₂ variable is a standardised level rather than a deviation from
2024, so the channel re-levels at the join between fitted history and projection. Direction
after roughly 2040 is credible; timing before then is not.

## Files

```
index.html                              the dashboard
trade_dashboard_small_<layer>.json.gz   pre-aggregated series (world, exporter, importer, product)
trade_dashboard_meta_<layer>.json.gz    dropdown lists and supported country pairs
features_<layer>.csv                    GDP, GDP per capita, population, CO₂ per capita
data_<layer>/flows/<EXP>/<IMP>.json.gz  one file per country pair, loaded on demand
models_manifest.json                    layer registry
```

Trade values are stored in **thousands of constant 2015 USD**; the dashboard multiplies by
1000 for display.

## Running locally

Gzip decoding does not work from `file://`, so serve over HTTP:

```bash
python -m http.server 8000
# then open http://localhost:8000
```

## Project

AMIGDALA (Horizon Europe). Data: BACI trade, SSP2 macro, TIAM-ECN emission pathways,
EXIOMOD GDP scenarios.
