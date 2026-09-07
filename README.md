# Multi-Curve SOFR Bootstrapper & Fixed Income Risk Engine (`sofr-curve-engine`)

A high-performance, vectorized fixed income pricing and term-structure risk engine implementing post-LIBOR multi-curve frameworks (SOFR OIS discounting and curve construction) built strictly on modern Python 3.12+ and SIMD-aligned NumPy linear algebra.

---

## 1. Architectural & Engineering Directives

- **Runtime Target:** Python 3.12+ (isolated deterministic numerical execution profiles).
- **Strict Typing:** PEP 585/604 lowercase generics and pipe syntax (`float | None`), PEP 646 / `numpy.typing.NDArray[np.float64]` for all array interfaces to eliminate silent dimension/casting bugs.
- **Anti-Loop Mandate:** Zero explicit procedural loops (`for`, `while`) or `pandas.apply()` across pricing, cash flow scheduling, and sensitivity grids.
- **Memory Profiling:** Zero-copy contiguous C-arrays (`order='C'`), explicit zero-copy extraction via `.to_numpy(dtype=np.float64, copy=False)`, and strict separation between $O(1)$ scalar metrics and $O(N \times K)$ tenor sensitivity matrices.
- **Decoupled Engine Core:** Stateless calculation core decoupled from ingestion, file I/O, and reporting layers.

---

## 2. Quantitative Mechanics & Scope

### A. Curve Construction & Multi-Curve Framework
- **Discount Factor Bootstrapping:** Solve for discount factors $P(0, T)$ from short-end cash rates, Fed Funds/SOFR basis, and vanilla SOFR Overnight Index Swaps (OIS).
- **Interpolation Schemes:** Monotone convex spline and piecewise log-linear zero-rate interpolation guaranteeing positive forward rates $f(t; T_1, T_2) > 0$.
- **Objective Function & Root Finding:** Vectorized root-solving (Newton-Raphson / Brent's method via SciPy/NumPy) minimizing swap net present value (NPV) residuals across pillar tenors.

### B. Valuation & Sensitivity Linear Algebra
- **Matrix Cash Flow Valuation:** Cash flow schedules modeled as structured arrays / 2D matrices where portfolio NPV is evaluated strictly via native dot products:
  $$\text{NPV} = \mathbf{P}^T \mathbf{C}$$
  where $\mathbf{P}$ is the vector of discount factors and $\mathbf{C}$ represents asset cash flow projections.
- **Duration & Convexity Decomposition:**
  - Analytical and bump-and-revalue DV01 (Dollar Value of a 01).
  - Modified Duration and Macauley Duration.
  - Key Rate Duration (KRD) vectors across standardized pillar tenors: 1M, 3M, 6M, 1Y, 2Y, 3Y, 5Y, 7Y, 10Y, 20Y, 30Y.
  - Effective Convexity matrices capturing non-parallel curve twists, steepening, and flattening modes.

---

## 3. Directory Layout

```text
sofr-curve-engine/
├── pyproject.toml
├── README.md
├── src/
│   └── sofr_curve/
│       ├── __init__.py
│       ├── types.py          # Custom TypeAliases, NDArray annotations
│       ├── conventions.py    # Day count (ACT/360, ACT/365, 30/360) and roll rules
│       ├── bootstrap.py      # Multi-curve bootstrapping and interpolation
│       ├── instruments.py    # Zero-coupon, fixed coupon bonds, vanilla OIS
│       └── sensitivities.py  # DV01, Key Rate Duration, Convexity engine
└── tests/
    ├── test_bootstrapper.py
    └── test_sensitivities.py
```
