---
name: bayesian-pymc
description: build, review, debug, and explain bayesian analyses in python using pymc, arviz, altair, pandas, and numpy. use when the user wants to create a new bayesian workflow from a dataset or research question, translate frequentist work into bayesian form, inspect or improve an existing pymc model or notebook, interpret priors/posteriors/predictive checks, compare models, or produce teaching-oriented explanations in the style of statistical rethinking with explicit generative assumptions, causal reasoning, and workflow discipline.
---

# Bayesian PyMC

Use this skill to produce practical Bayesian analysis work in a Statistical Rethinking inspired style.

## Core stance

- Start from the scientific or decision question, not from the estimator.
- Prefer explicit generative assumptions over opaque formula-chasing.
- Make priors part of the model design, not an afterthought.
- Treat causal assumptions separately from statistical assumptions; be explicit about both.
- Build models in layers and check each layer before moving on.
- Default to posterior predictive checking, parameter interpretation on meaningful scales, and honest discussion of uncertainty.
- Prefer clear code and plots that teach the model.

## Default stack

Prefer these libraries unless the user already committed to something nearby:

- `pymc` for modeling and inference
- `arviz` for diagnostics, posterior summaries, and predictive checks
- `altair` for explanatory charts
- `pandas` and `numpy` for data handling

Avoid introducing extra frameworks unless they are required by the task.

## Supported inputs

This skill should flex to the user's context. Common inputs include:

- a CSV, parquet file, or tabular dataset
- a pandas DataFrame or schema description
- a research question in plain language
- existing Python or notebook code
- a frequentist model that needs Bayesian reformulation
- a request for interpretation, critique, teaching notes, or model comparison

When the input is ambiguous, infer the most useful artifact to produce from the request and state your assumptions briefly.

## Supported outputs

Choose the output shape that best matches the request. Common outputs include:

- a full PyMC script
- a notebook-style analysis outline
- a revised or debugged model
- prior design notes and justification
- posterior summaries and interpretation
- posterior predictive checks
- model comparison guidance
- Altair charts
- teaching-oriented explanations and causal critique

## Workflow decision tree

1. **New analysis from data or a question** → follow the New Analysis workflow.
2. **Existing PyMC code needs review/debugging/improvement** → follow the Model Review workflow.
3. **User wants explanation, critique, or translation** → follow the Explanation and Translation workflow.

## New Analysis workflow

1. Define the estimand or question in plain language.
2. Identify outcome type and plausible data-generating process.
3. Write the model as explicit assumptions:
   - likelihood
   - linear/nonlinear structure
   - priors
   - grouping, measurement, censoring, missingness, or exposure if relevant
4. Check scale before coding:
   - standardize or center predictors when helpful
   - choose priors on interpretable scales
   - say what would count as implausible predictions
5. Draft the PyMC model with named coordinates and deterministic quantities when they help interpretation.
6. Fit with defaults that are likely to work, but adjust when diagnostics indicate trouble.
7. Report diagnostics and posterior summaries.
8. Run posterior predictive checks.
9. If useful, compare against simpler or competing models.
10. End with substantive interpretation, not only coefficient recitation.

## Model Review workflow

When reviewing existing PyMC code:

1. Restate the implied generative model in plain English.
2. Check whether the likelihood matches the outcome.
3. Inspect priors for scale, regularization, and accidental informativeness.
4. Look for parameterization issues:
   - centered vs non-centered hierarchies
   - redundant parameters
   - unidentified interactions
   - overly diffuse priors
5. Check diagnostics conceptually and, when available, from ArviZ:
   - divergences
   - `r_hat`
   - effective sample size
   - energy / geometry warnings
6. Recommend the smallest high-leverage fixes first.
7. Preserve the user's modeling intent unless it is clearly flawed.

## Explanation and Translation workflow

For explanation-heavy tasks:

1. Explain the model as a story about how the data could arise.
2. Translate parameters into domain language.
3. Distinguish clearly between:
   - prior assumptions
   - posterior update
   - predictive implications
   - causal claims
4. When translating from frequentist work, map each component:
   - likelihood
   - linear predictor
   - regularization / priors
   - inferential target
5. Prefer short worked examples over abstract lectures.

## Priors

Use weakly regularizing priors by default, not flat priors.

Guidelines:

- Put priors on meaningful scales. Mention the scale explicitly.
- Standardize inputs when it makes priors easier to reason about.
- For intercepts, use domain knowledge about plausible baseline outcomes.
- For slopes, prefer shrinkage toward small effects unless the context argues otherwise.
- For scale parameters, use positive priors that rule out absurd variability.
- For probabilities and logits, think through the implied outcome range.
- If priors are controversial, show at least one sensible alternative and explain the tradeoff.

When relevant, perform or describe a prior predictive check before fitting.

## Diagnostics and model checking

Always mention or perform the most relevant checks.

Minimum default checks:

- whether the sampler mixed adequately
- whether there are divergences or geometry problems
- whether posterior predictions resemble observed structure
- whether inferences are stable under a simpler comparison model

If a model is complex, recommend building it progressively and validating sub-models first.

## Visualization

Prefer Altair for user-facing charts.

Default chart priorities:

- prior vs posterior comparison when priors are central
- posterior distributions with intervals
- observed vs posterior predictive distributions
- conditional effects / marginal predictions
- group-level shrinkage visuals for hierarchical models

Charts should teach one idea each. Avoid overloaded plots.

See `references/visualization-patterns.md`.

## Coding conventions

- Use modern `pymc` APIs, not PyMC3-era style, unless the user is maintaining old code.
- Prefer explicit variable names over single-letter symbols in code.
- Use `coords` and `dims` for grouped models when helpful.
- Keep model code compact, but separate data prep, model definition, fitting, checking, and plotting.
- Compute transformed quantities with `pm.Deterministic` only when they aid interpretation.
- Use ArviZ objects and summaries rather than ad hoc array slicing when possible.
- Do not present code without explaining why the likelihood and priors fit the problem.

## Output template

Use this default structure unless the request clearly calls for another format:

### 1. Problem framing
- research or decision question
- estimand or target quantity
- key assumptions

### 2. Model design
- likelihood
- predictor structure
- priors and rationale

### 3. PyMC implementation
- code

### 4. Checks
- diagnostics
- posterior predictive checks
- comparison to simpler alternatives when relevant

### 5. Interpretation
- substantive meaning
- uncertainty
- limitations and causal caveats

## Style to emulate

Adopt these habits associated with Statistical Rethinking inspired practice:

- think in generative stories
- emphasize causal structure and confounding awareness
- explain why controls can help or harm
- use simple models as baselines for more complex ones
- treat workflow and quality assurance as part of the analysis, not cleanup afterward
- write for understanding, not ritual correctness

## Resources

- For practical workflow guidance, read `references/workflow.md`.
- For reusable PyMC patterns, read `references/pymc-patterns.md`.
- For plotting guidance, read `references/visualization-patterns.md`.
