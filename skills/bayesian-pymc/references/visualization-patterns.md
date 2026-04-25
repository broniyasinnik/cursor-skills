# Visualization patterns

Prefer Altair for final plots.

## Principles

- one plot, one message
- show uncertainty directly
- label quantities in domain language
- compare observed and predicted data whenever possible

## Useful chart types

### Posterior interval plot
Use for a small set of parameters or contrasts.

### Prior vs posterior density plot
Use when prior sensitivity or regularization is central.

### Observed vs posterior predictive overlay
Use to show whether the model reproduces the data's shape.

### Conditional effect ribbon
Use for regression-type models to show expected outcome across a predictor with uncertainty bands.

### Group shrinkage comparison
Use to show partial pooling by comparing raw group estimates to hierarchical estimates.

## Altair defaults

- tidy data into long format before plotting
- use faceting sparingly
- avoid decorative color choices unless they encode meaning
- do not overload the user with every posterior draw on one chart

## What to say with each plot

Each plot should be paired with one sentence that answers:

- what should the user notice?
- why does it matter for the model or the decision?
