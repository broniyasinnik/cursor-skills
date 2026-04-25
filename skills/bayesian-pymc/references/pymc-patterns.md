# PyMC patterns

## Skeleton for a new model

```python
import numpy as np
import pandas as pd
import pymc as pm
import arviz as az

coords = {"obs": np.arange(len(df))}

with pm.Model(coords=coords) as model:
    x = pm.Data("x", df["x"].to_numpy(), dims="obs")
    y = pm.Data("y", df["y"].to_numpy(), dims="obs")

    alpha = pm.Normal("alpha", mu=0, sigma=1)
    beta = pm.Normal("beta", mu=0, sigma=1)
    sigma = pm.Exponential("sigma", 1)

    mu = alpha + beta * x

    pm.Normal("y_obs", mu=mu, sigma=sigma, observed=y, dims="obs")

    idata = pm.sample()
    idata = pm.sample_posterior_predictive(idata, extend_inferencedata=True)

az.summary(idata, var_names=["alpha", "beta", "sigma"])
```

Adapt the likelihood before adapting anything else.

## Likelihood selection hints

- continuous roughly symmetric outcome → `pm.Normal`
- positive skewed outcome → often model on log scale or consider Gamma / LogNormal
- binary outcome → `pm.Bernoulli` with logit link
- count outcome → `pm.Poisson` or `pm.NegativeBinomial` when overdispersed
- ordinal outcome → use ordered thresholds
- proportions with varying denominators → binomial rather than transformed rates when possible

## Hierarchical pattern

Use partial pooling when units repeat and unit-level estimates are noisy.

```python
coords = {
    "obs": np.arange(len(df)),
    "group": df["group"].unique(),
}

group_idx = pd.Categorical(df["group"], categories=coords["group"]).codes

with pm.Model(coords=coords) as model:
    x = pm.Data("x", df["x"].to_numpy(), dims="obs")
    y = pm.Data("y", df["y"].to_numpy(), dims="obs")
    g = pm.Data("g", group_idx, dims="obs")

    alpha_bar = pm.Normal("alpha_bar", 0, 1)
    alpha_sd = pm.Exponential("alpha_sd", 1)
    alpha_offset = pm.Normal("alpha_offset", 0, 1, dims="group")
    alpha = pm.Deterministic("alpha", alpha_bar + alpha_offset * alpha_sd, dims="group")

    beta = pm.Normal("beta", 0, 1)
    sigma = pm.Exponential("sigma", 1)
    mu = alpha[g] + beta * x

    pm.Normal("y_obs", mu=mu, sigma=sigma, observed=y, dims="obs")
```

Prefer a non-centered parameterization when group-level variance is weakly identified or sampling is difficult.

## Diagnostics checklist

Look for:

- zero divergences when feasible
- `r_hat` near 1.00
- adequate effective sample sizes
- sensible posterior predictive behavior

When diagnostics fail, try in this order:

1. inspect priors and scales
2. simplify the model
3. reparameterize
4. increase `target_accept` if geometry still needs help

## Translation pattern from frequentist to bayesian

When the user supplies a frequentist model, map it into:

- same likelihood family if appropriate
- explicit priors for each parameter block
- posterior summaries instead of p-values
- predictive checks instead of coefficient-only validation

Do not preserve a bad likelihood just because the legacy code used it.
