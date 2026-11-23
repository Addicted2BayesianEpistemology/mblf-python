# mblf

Python package for BMLF, the Bayesian latent factors approach for multimodal functional data. Several functional modes
are observed on the same subjects, each on its own grid and possibly with missing values; shared latent factors link
the modes and allow one mode to be predicted from the others. The posterior is sampled with Stan through
`cmdstanpy`.

## Installation

Python 3.9 or later.

```bash
pip install "git+https://github.com/Addicted2BayesianEpistemology/mblf-python@v1.0.0"
python -c "import cmdstanpy; cmdstanpy.install_cmdstan()"     # if CmdStan is not installed yet
```

The Stan programs are compiled at first use and cached in `~/.cache/mblf`. CmdStan cannot compile in a folder whose
path contains spaces; in that case set `MBLF_CACHE_DIR` to another folder.

## Example

Two modes driven by one latent factor; mode 2 of ten new subjects is predicted from their mode 1. `n_components` is
a cap: the prior shrinks the factors that the data do not need.

```python
import numpy as np
import mblf

rng = np.random.default_rng(1)
t = np.linspace(0, 1, 30)
eta = rng.normal(size=50)
Y1 = np.outer(np.sin(2 * np.pi * t), eta) + rng.normal(scale=0.2, size=(30, 50))
Y2 = np.outer(np.cos(2 * np.pi * t), eta) + rng.normal(scale=0.2, size=(30, 50))

modes = []
for Y in (Y1, Y2):
    fd = mblf.FunctionalData(t, Y[:, :40])
    fd.set_gp_basis(mblf.GPBasis("squared_exponential", energy=0.99, weights="trapezoid", estimate=True))
    modes.append(fd)
fit = mblf.MvtBLF(n_components=2).fit(modes, iter_sampling=2000)
print(fit.diagnostics())
pred = fit.predict(target=2, sources={1: Y1[:, 40:]})
print(pred["mean"].shape)
```

Each column of a data array is a subject and each row a location of the grid; `NaN` marks a value that is not
observed. Modes are numbered from 1.

## Contents

- `FunctionalData`: the data of a mode and its basis (`set_gp_basis`, or `set_basis` for any basis matrix).
- `GPBasis`: a Gaussian-process eigenbasis.
- `MvtBLF`: the model; its methods sample the posterior (`fit`) and return the loadings (`loadings`,
  `functional_loadings`, `aligned_loadings`), predictions of unobserved modes (`predict`), the covariance of the
  latent curves (`covariance`), convergence diagnostics (`diagnostics`) and PSIS-LOO over subjects (`loo`).
- `one_se_rule`: the choice of the number of factors.
- `fit_ml`: maximum-likelihood estimation.
- `effective_dimension`, `projection_diagnostics`, `heldout_scores`, `check_likelihood`: the diagnostics of the
  secondary studies of the paper.

The arguments and the value of each function are described in its docstring.

## Citation

B. Ursino, I. Epifani and F. Trovò. *A Bayesian latent factors approach for multimodal functional data.*

## Licence

MIT, see `LICENSE`.

The same model in R is [mblf-r](https://github.com/Addicted2BayesianEpistemology/mblf-r); the comparator methods of
the paper are in [mblfcomp-r](https://github.com/Addicted2BayesianEpistemology/mblfcomp-r) and
[mblfcomp-python](https://github.com/Addicted2BayesianEpistemology/mblfcomp-python), and the analyses of the paper
in [mblf-paper](https://github.com/Addicted2BayesianEpistemology/mblf-paper).
