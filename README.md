# Signal Processing over Product DAGs: Shift Operator and Filters

By Sundeep Prabhakar Chepuri, Antonio G. Marques, Maulik Devmurari, and Gonzalo Mateos

Claude AI was used to assist in generating this code

Code and data accompanying the aforementioned paper. 

# Product DAG Low-Pass Denoising

A product DAG low-pass filter
$\hat{\mathbf S}=\mathbf W_1(\tilde{\mathbf H}\odot\mathbf C)\mathbf W_2^\top$ is fit two ways —
**unconstrained** ($\tilde{\mathbf H}$, $n_1n_2$ parameters, matrix-free conjugate gradients) and **rank-1 separable** ($\tilde{\mathbf H}=\mathbf h_1\mathbf h_2^\top$, $n_1+n_2$ parameters, the proposed alternating least squares / ALS) — and both are compared against a **no-filter** baseline, on **synthetic** data and on **real** weekly water-quality measurements from the River Thames.

## Contents

- `product_lowpass_als_thames.ipynb` — the real-data experiment: builds the river-flow DAG,     prunes it to the sites the real CSV covers, loads the determinand, fits and cross-validates the
  filters, and produces every figure/table used in the paper's real-data section.
- `THAMES_data.csv` — the CEH *Thames Initiative* weekly water-quality record, 2009–2023 (DOI
  [`10.5285/cf10ea9a-a249-4074-ac0c-e0c3079e5e45`](https://doi.org/10.5285/cf10ea9a-a249-4074-ac0c-e0c3079e5e45)).
- `product_lowpass_als_synthetic.ipynb` — the synthetic-data experiment: two random factor DAGs,
  a separable low-pass signal model, white noise, and the same three-way comparison — produces
  every figure/table used in the paper's synthetic-data section.

## Running

Requires Python 3 with `numpy`, `scipy`, `networkx`, and `matplotlib` (the Thames notebook also
needs `pandas`). Open either notebook and run all cells top to bottom (`THAMES_data.csv` must be
in the same directory as `product_lowpass_als_thames.ipynb`, or point the `THAMES_CSV`
environment variable at it). Runtime is under a minute end to end for each notebook.

## `product_lowpass_als_thames.ipynb` — real data

### What the notebook does

1. **Estimators** (Sec. 0): the analysis/synthesis operators and the three fitting routines
   (`fit_full`, `fit_als`, `fit_axis`) — the core method, shared with the synthetic-data notebook.
2. **River DAG** (Sec. 1): the 20-node design network (7 main-stem stations, 13 tributaries) with
   the weekly time chain.
3. **Real data loading and DAG pruning** (Sec. 2–2b): maps the CSV's actual site names onto the
   design, drops the 5 design nodes with no matching site (rerouting edges through the one
   non-leaf omission to preserve downstream orientation), and loads **total dissolved phosphorus**
   — the determinand used throughout, selected offline as detailed below.
4. **Leave-one-year-out cross-validation** (Sec. 4–4b): every usable year is held out once, the
   filters are fit on the rest, and held-out NMSE is reported for `no filter`, `separable (ALS)`,
   and `unconstrained`, swept over SNR $\in\{4,6,7,10\}$ dB (Sec. 4b reproduces the paper's table
   exactly).
5. **Visualizations** (Sec. 5): clean/noisy/denoised field heatmaps, the fitted separable
   response, and the DAG coloured by one week's values.
6. **Computational cost** (Sec. 6): factored vs. dense filter application time vs. $N$.

### Determinand selection

Missing entries in `THAMES_data.csv` are common and vary a lot by determinand and by year (some
tributary sites stop reporting several determinands entirely from 2019 onward). Total dissolved
phosphorus was selected via a two-step, pre-specified procedure (not tuned to the outcome):

**Least missing**: computed on the raw weekly (site × week) pivot, before interpolation, over
   the 15 mapped sites. Total dissolved phosphorus is tied with dissolved fluoride/chloride/
   sulphate to within 9 cells out of 11,940 — a negligible difference.

### Missing-data handling

A year is only used if *every* one of the 15 mapped sites has at least 20 raw (pre-interpolation)
weekly readings that year (`MIN_READINGS` in Sec. 2) — otherwise a site with near-zero real
readings would need its whole column fabricated to reach 52 weeks, which is invention, not
denoising. This excludes 2009 (ramp-up), 2018 (a mid-year coverage change), and 2019–2023 (several
tributary sites report zero readings for many determinands) at the pruned-DAG level, leaving 8
usable years, 2010–2017. Within those years, 94% of site-week cells are directly measured and 6%
are filled by linear interpolation within the same calendar year; nothing is fabricated.


## `product_lowpass_als_synthetic.ipynb` — synthetic data

### What the notebook does

1. **Estimators** (Sec. 0): the same analysis/synthesis operators and fitting routines
   (`fit_full`, `fit_als`, `fit_axis`) as the Thames notebook, plus `rdag` — the random-DAG
   generator used only here.
2. **Two random factor DAGs** (Sec. 1): $G_1$ (space, $N_1=6$ nodes, edge probability $p=0.45$)
   and $G_2$ (stage, $N_2=5$ nodes, $p=0.5$), each edge weight drawn uniformly from $[0.3,0.7]$,
   with any parentless node attached to a random predecessor to guarantee weak connectivity —
   giving a product DAG $G_\times$ with $N=N_1N_2=30$ nodes.
3. **Visualization** (Sec. 2): $G_1$, $G_2$, and the product DAG built from the operator adjacency
   $A_\times=A_1\otimes I+I\otimes A_2-A_1\otimes A_2$.
4. **Low-pass signal, white noise** (Sec. 3): clean signals with a separable low-pass power
   spectrum $\mathbb E|C_{ij}|^2=\rho_1^{\mathrm{depth}_1(i)}\rho_2^{\mathrm{depth}_2(j)}$
   ($\rho_1=0.30,\rho_2=0.34$), corrupted by white Gaussian noise; filters are fit on 60 training
   pairs and evaluated on 80 held-out pairs at 6 dB, reporting held-out NMSE for `no filter`,
   `separable (ALS)`, and `unconstrained`.
5. **Low-pass heat map** (Sec. 4): one held-out field at 4 dB (clean / noisy / ALS-denoised /
   unconstrained-denoised) as node colours on the product DAG.
6. **Computational cost** (Sec. 6): factored vs. dense filter application time vs. $N$, swept up
   to a $60\times60$ product DAG ($N=3600$).


