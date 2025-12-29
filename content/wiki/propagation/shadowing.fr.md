+++
title = "Large-Scale Fading: Shadowing"
date = 2025-12-28
math = true
summary = "An overview of shadowing effects in wireless channels and log-normal modeling."
+++

## 1) What is Shadow Fading (and what it is not)

Shadow fading (often called *shadowing* or *SF*) represents **medium-scale fluctuations** of the received signal strength produced by obstacles in the propagation path (buildings, trees, street furniture, etc.). It varies **slowly with position**, typically over distances comparable to the size of the dominant obstructions.

It sits between:
- **Path loss (PL):** the deterministic/mean trend vs. distance and scenario (UMa/UMi, LoS/NLoS, etc.).
- **Fast fading (FF):** rapid fluctuations due to multipath interference (small-scale), varying over fractions of a wavelength and across frequency/time/antennas.

A practical mental model:
- PL sets the *slope* (distance-dependent mean).
- SF adds a *slow random offset* around that slope (spatially smooth).
- FF adds *rapid ripples* around the local mean.

---

## 2) Standard log-normal model (SF in dB is Gaussian)

In wireless communications, shadowing is commonly modeled as **log-normal in linear scale**, which is equivalent to a **Gaussian random variable in dB**:

$$
P_r(d) = P_t - PL(d) + X_\sigma
$$

where:
- $P_t$ is the transmitted power (in dB units, e.g., dBm),
- $PL(d)$ is the path loss at distance $d$ (in dB),
- $X_\sigma \sim \mathcal{N}(0,\sigma^2)$ is the shadow fading term in dB.

In linear scale, if we define the multiplicative shadowing factor

$$
S = 10^{X_\sigma/10},
$$

then $S$ is log-normal.

---

## 3) Spatial correlation: correlation distance matters

Shadow fading is not i.i.d. in space: **nearby positions share similar obstructions**, so SF is spatially correlated.

A widely used model is an exponential autocorrelation:

$$
\mathrm{Corr}\{X_\sigma(\mathbf{u}), X_\sigma(\mathbf{v})\} =
\exp\left(-\frac{\|\mathbf{u}-\mathbf{v}\|}{d_\mathrm{corr}}\right)
$$





where $d_\mathrm{corr}$ is the **correlation distance** (scenario-dependent).

Interpretation:
- If $\|\mathbf{u}-\mathbf{v}\| \ll d_\mathrm{corr}$, shadowing is strongly correlated.
- If $\|\mathbf{u}-\mathbf{v}\| \gg d_\mathrm{corr}$, shadowing becomes almost independent.

This is essential when you want realistic maps: SF should look like a **smooth random field**, not pixel-level noise.

---

## 4) Shadowing as an LSP in 3GPP TR 38.901

In TR 38.901, SF is part of the **Large-Scale Parameters (LSPs)** family (together with DS, ASD, ASA, K, ZSD, ZSA, etc.).

Two key ideas:

### 4.1 Cross-correlation between different LSP components (per link)
For a given BS–UT link, the LSP vector is sampled so that components are **mutually correlated** (e.g., SF correlated with DS, etc.). This is typically done by starting from a Gaussian vector and applying a correlation matrix (e.g., via Cholesky). Sionna follows this idea when generating the LSP realization per link.

### 4.2 Spatial correlation of each LSP component (across UTs)
Across multiple UT positions, each LSP component is also **spatially autocorrelated**, such that UTs near each other share similar LSP values (including SF). This is what makes channel snapshots coherent over space.

---

## 5) How Sionna puts everything together (PL, SF, and h_freq)

Sionna internally builds a *Scenario* (UMaScenario/UMiScenario/…) and exposes a channel model `cm`. The scenario contains geometry, LoS/NLoS states, and the TR 38.901 parameter tables (often stored in JSON internally).

### 5.1 Model path loss (TR 38.901) vs. sampled path loss (with O2I)
- `cm._scenario.basic_pathloss` gives the **basic path loss** from the scenario formulas (TR 38.901).
- `cm._lsp_sampler.sample_pathloss()` gives the **full path loss** used by the LSP sampler (basic PL + possible O2I). For outdoor-only simulations, O2I is typically zero, so both are often close.

### 5.2 LSP realization (including SF) conditions the small-scale fading
Sionna generates one realization of the LSP vector per BS–UT link and stores it (e.g., in `cm._lsp.sf`, `cm._lsp.ds`, etc.). This realization is what conditions the statistics of the fast fading process.

Important conceptual point:
- LSPs are the **slow variables** (random but slowly varying over space).
- Fast fading is a **random process conditioned on LSPs**: you can generate many FF realizations while keeping the same LSP realization fixed, until you move far enough or re-sample LSPs.

### 5.3 h_freq already embeds PL + SF + FF
After sampling PL and SF, Sionna scales the small-scale channel with a gain factor (often denoted conceptually as $g$), producing a frequency-domain channel (`h_freq` or `h_full`) that **already contains PL + SF + FF**.

---

## 6) `sample_pathloss()` vs. `get_pathloss(h_freq)` (very common confusion)

Sionna gives you two “path loss-like” objects:

### 6.1 Model path loss (from TR 38.901)

$$
PL_{\mathrm{model}} \approx \text{sample\\_pathloss}(cm)
$$

In Sionna: `cm._lsp_sampler.sample_pathloss()`




This is the loss used by the model to scale the channel (Step 12 in the TR 38.901 flow).

### 6.2 Effective path loss estimated from the realized channel
`get_pathloss(h_freq)` computes an effective loss by:
1) computing a mean channel power (averaging over subcarriers and antennas), and  
2) converting it to dB as a negative log measure.

Conceptually:

$$
PL_\text{eff}(h_\text{freq}) \triangleq -10\log_{10}\left(\mathbb{E}\{|h_\text{freq}|^2\}\right)
$$

(with the expectation implemented by a finite average across dimensions).

Key implication:
- $PL_\text{eff}$ includes **PL + SF + residual effects of FF**, and can also reflect array/pattern/orientation effects that sit inside the realized channel.

So, a useful approximation is:

$$
PL_\text{eff} \approx PL_\text{model} + SF + \varepsilon
$$

where $\varepsilon$ is not “just fast fading”; it includes finite-averaging error, residual small-scale energy, antenna pattern/orientation effects, and OFDM normalization offsets.

---

## 7) How to isolate Shadow Fading (practically)

If your goal is a **shadowing map** (slow variations), the strategy is:

1) Compute $PL_\text{model}$ from the model:
   - `pl_db = cm._lsp_sampler.sample_pathloss()` (or `basic_pathloss` if you explicitly want the basic PL)
2) Estimate $PL_\text{eff}$ from the realized channel:
   - `pl_eff_db = get_pathloss(h_freq)`
3) Take the difference:

$$
SF_\text{est} \approx PL_\text{eff} - PL_\text{model}
$$

Caveat (important): the difference will be “SF + residuals” unless you average enough independent samples to suppress FF.

---

## 8) Fast fading: narrowband vs wideband power (a robust trick)

If you only have access to `h_freq`, one robust way to characterize FF is to compare:
- **Narrowband power** (one resource element, average over antennas)
- **Wideband/local mean power** (average over time/frequency/antennas in a window that spans many independent samples, but stays inside a region where LSPs are constant)

Define:

$$
P_\text{NB}(t_0,f_0)=\frac{1}{N}\sum_{n=1}^N |h(t_0,f_0,n)|^2
$$

and

$$
P_\text{WB}=\frac{1}{TFN}\sum_{t\in\mathcal{T}}\sum_{f\in\mathcal{F}}\sum_{n=1}^N |h(t,f,n)|^2
$$

Then:

$$
FF_\text{dB}(t_0,f_0)=10\log_{10}\left(\frac{P_\text{NB}(t_0,f_0)}{P_\text{WB}}\right)
$$

Why this is nice:
- The ratio cancels the slow gain $g(\mathbf{u})$ (PL + SF), leaving a dimensionless measure of “how above/below the local mean” a given RE is.

Condition for it to behave as true FF:
- $P_\text{WB}$ should average enough independent samples (wider than coherence bandwidth/time, low antenna correlation), **without moving too far** such that LSPs change.

---

## 9) Practical checklist (so your maps look physically consistent)

- If you plot “shadowing”, make sure you’re not accidentally plotting a **residual of averaged fast fading**.
- If you estimate from `h_freq`, increase averaging (subcarriers, OFDM symbols, antennas, Monte Carlo) until the residual becomes smooth.
- Keep in mind: antenna pattern/orientation can leak into $PL_\text{eff}$ when estimated from `h_freq`.
- When comparing LoS vs NLoS, expect different SF variance and different spatial behavior (scenario-dependent).

---

## 10) What to explore next

- **Correlation distance:** estimate $d_\mathrm{corr}$ empirically from SF realizations across UT positions.
- **3GPP modeling:** contrast SF statistics between UMa and UMi (LoS/NLoS).
- **Validation with Sionna internals:** use `cm._lsp.sf` as the reference SF realization and compare with your SF estimate from `h_freq`.
