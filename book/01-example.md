# Example episode

:::{admonition} Overview
:class: note
**Questions**

* FIXME: what question does this episode answer?

**Objectives**

* FIXME: what will learners be able to do after this episode?
:::

## Writing content

Pages are written in [MyST Markdown](https://jupyterbook.org/en/stable/content/myst.html).
Code can be shown with fenced code blocks:

```python
import numpy as np

rng = np.random.default_rng()
print(rng.normal(size=5))
```

Figures placed in this directory can be included with the `figure` directive:

```{figure} hsf-logo.png
:width: 200px
:name: hsf-logo

The HSF logo (replace this with your own figures).
```

You can also cite references from `references.bib`, like this evidence for
predictive coding in auditory cortex {cite}`holdgraf_evidence_2014`.

:::{admonition} Exercise: your first change
:class: tip
FIXME: describe a short exercise for the learners here.
:::

::::{admonition} Solution
:class: dropdown
FIXME: put the solution to the exercise here.

:::{admonition} Note
Nested admonitions need more colons on the outer fence than on the inner one.
:::
::::

:::{admonition} Key Points
:class: important
* FIXME: first key point of this episode
* FIXME: second key point of this episode
:::

## References

```{bibliography}
```
