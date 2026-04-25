# Bayesian workflow guide

This reference captures the operating principles behind the skill.

## 1. Start with the question

Before coding, state:

- what quantity matters
- what would count as an informative answer
- which variables are causes, proxies, controls, or post-treatment variables

Avoid jumping straight to a regression formula.

## 2. Tell the generative story

Write a compact narrative:

- how outcomes arise
- what variation is expected
- where constraints, ceilings, counts, categories, or grouping enter

Then convert that story into likelihood plus structure.

## 3. Design priors on the scale of consequences

Choose priors by asking what outcomes they imply. Good practice:

- reason on the scale of the outcome and predictors
- use standardization if it makes prior reasoning easier
- regularize toward realistic effects, not zero-information fantasy

## 4. Build models progressively

For complex models, do not jump to the final hierarchy immediately.

Suggested sequence:

1. intercept-only baseline
2. simple predictor version
3. add interactions or nonlinearity
4. add varying effects
5. add measurement, missingness, or special structure

At each stage, inspect whether the model learned something coherent.

## 5. Check predictions, not only parameters

Posterior predictive checks should answer: if this model were true, would data like these be ordinary?

Useful checks:

- raw distribution shape
- means / rates by group
- tail behavior
- zero inflation or overdispersion
- trend curvature
- calibration of uncertainty

## 6. Distinguish explanation from causation

A statistical model can describe associations without identifying a causal effect.

When causal language appears:

- say what assumptions are required
- name plausible confounds
- avoid controlling for variables that may be mediators or colliders
- prefer a brief DAG-style explanation when it helps

## 7. Compare models as part of learning

Model comparison is not only a winner-picking exercise. Use it to learn:

- whether extra complexity earns predictive or explanatory value
- whether a simpler baseline already answers the question
- whether conclusions depend on delicate assumptions

## 8. End with a domain-level interpretation

Do not stop at coefficients. Explain:

- what changed in expected outcome or probability
- how certain the conclusion is
- what predictions would differ in the world
- what assumptions could break the conclusion
