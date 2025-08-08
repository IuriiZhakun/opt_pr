# opt\_pr — Options Pricing & Risk Toolkit (modular, calendar‑aware)

**opt\_pr** (Option’s Prime Radiant 😉) is a Python toolkit for **options pricing and scenario analysis** designed to scale from notebooks to research backtests and production prototypes.

- **Today (v0.1):** American options on **equity index futures** with:

  - **CRR binomial** lattice + **Richardson acceleration** (accurate baseline)
  - **Longstaff–Schwartz (LSM) Monte Carlo** (structured for batching/GPU)
  - **Black–76** European reference (sanity/arbitrage checks)
  - **QuantLib 1.29** Barone–Adesi & Whaley (optional) with **calendar support**
  - **Calendar‑aware** time/steps (Actual/365 Fixed; steps aligned to **business days**)
  - **Plotly** helpers for interactive exploration

- **Near‑term roadmap:** broader product set and engines (see below).

Author: **Iurii Zhakun** ([yurij.zhacun@gmail.com](mailto\:yurij.zhacun@gmail.com))\
License: **BSD‑3‑Clause**

---

## Vision & roadmap

The package is built to expand beyond American futures options. Planned areas:

- **Products**

  - Equity & equity‑index **spot** options (Black–Scholes)
  - **Futures** options (current) across asset classes
  - **FX** (Garman–Kohlhagen), quanto overlays
  - Early‑exercise styles: **American**/**Bermudan**, **European** (current for reference)
  - Structured add‑ons: simple **barriers** (down‑the‑road)

- **Engines**

  - Trees (CRR/Trigeorgis/Leisen–Reimer), **Richardson** / **Andersen–Lake** accelerators
  - **LSM** (current), batched **GPU** variants (TensorFlow extra; PyTorch optional later)
  - Closed forms: **Black–Scholes / Black–76** (current), **BAW** (current via QuantLib)

- **Infrastructure**

  - **Calendar/day‑count** via QuantLib or pure‑Python fallback
  - Engine regression suite vs. QuantLib & known results
  - Scenario tooling: batched/broadcast inputs, grid utilities, result frames

*(If you have specific priorities—e.g., barrier features or FX first—open an issue and we’ll steer the roadmap.)*

---

## Installation (Poetry)

```bash
# Core install
poetry install

# With optional extras
poetry install --with plots,ql,tf
```

**Extras**

- `plots` → `plotly>=5.0` for visuals
- `ql`    → `QuantLib==1.29` for BAW & calendars
- `tf`    → `tensorflow>=2.15` for future GPU‑batched LSM

> Requires Python **3.11**.

---

## Quick start (current v0.1 capabilities)

```python
from datetime import date
from opt_pr import (
    yearfrac_act365_calendar, count_business_days,
    black76_dates, american_binomial_fut_richardson, american_lsm_fut,
)

F, K, r, vol, is_call = 5000.0, 5000.0, 0.04, 0.20, True
valuation = date(2025, 8, 8); expiry = date(2025, 9, 7); cal = "US"

# Calendar‑aware year fraction and step count
T = yearfrac_act365_calendar(valuation, expiry, cal)
steps = max(1200, count_business_days(valuation, expiry, cal))

# European reference (Black–76)
eu = black76_dates(F, K, r, vol, is_call, valuation, expiry, cal)

# American — binomial with Richardson
am_tree = american_binomial_fut_richardson(F, K, r, vol, T, is_call, steps)

# American — LSM Monte Carlo (coarser steps, many paths)
am_lsm = american_lsm_fut(F, K, r, vol, T, is_call, steps=min(steps, 120), paths=40_000)

print("EU (Black–76)", eu)
print("AM (Tree)  ", am_tree)
print("AM (LSM)   ", am_lsm)
```

With **QuantLib** (optional):

```python
from opt_pr import quantlib_american_fut_baw
price_ql = quantlib_american_fut_baw(F, K, r, vol, is_call, valuation, expiry, cal)
print("AM (QuantLib BAW)", price_ql)
```

---

## API overview (v0.1)

- `yearfrac_act365_calendar(valuation_date, expiry_date, calendar)` → float
- `count_business_days(valuation_date, expiry_date, calendar)` → int
- `black76(F, K, r, vol, T, is_call)` / `black76_dates(..., calendar)`
- `american_binomial_fut(F, K, r, vol, T, is_call, steps)`
- `american_binomial_fut_richardson(F, K, r, vol, T, is_call, steps)`
- `american_lsm_fut(F, K, r, vol, T, is_call, steps, paths, seed)`
- `quantlib_american_fut_baw(F, K, r, vol, is_call, valuation_date, expiry_date, calendar)`
- `sanity_checks(F, K, r, vol, is_call, valuation_date, expiry_date, calendar)`
- `plotting.price_vs_strike(...)`, `plotting.surface_F_vol(...)`, `plotting.slider_vol_over_strikes(...)`

> American pricers assume **futures** dynamics; in BSM we set **S := F, q := r** so the forward drift is 0.

---

## Design principles

- **Modular engines** with consistent signatures to enable scenario sweeps
- **Calendar‑aware** by default; **uniform Δt** for recombining lattices
- **Multiple methods** for the same product to catch calibration/model issues
- **Deterministic seeds** and reproducible numerics
- **Performance‑aware**: vectorized code, GPU‑ready Monte Carlo

---

## Development

```bash
# Lint & format
poetry run ruff check opt_pr
poetry run black opt_pr

# Tests
poetry run pytest -q
```

### Project layout

```
opt_pr/
  __init__.py
  timeconv.py         # calendar + ACT/365F
  black76.py          # European (Black–76)
  binomial.py         # CRR + Richardson
  lsm.py              # LSM Monte Carlo
  quantlib_baw.py     # QuantLib BAW (optional)
  sanity.py           # arbitrage & monotonicity checks
  plotting.py         # Plotly helpers (optional)
examples/
  demo_es.py
```

---

## License

BSD‑3‑Clause

## Acknowledgements

- QuantLib project & contributors.
- Classic references: Black (1976), Barone–Adesi & Whaley (1987), Longstaff & Schwartz (2001).

