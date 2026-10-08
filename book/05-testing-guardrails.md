# Testing and Guardrails: Validating AI-Generated Code

:::{admonition} Overview
:class: note
**Questions**

* How can we trust LLM-generated code for scientific and physics applications?
* How do we prevent AI models from silently hallucinating math or physics logic?

**Objectives**

* Understand why Test-Driven Development (TDD) is highly recommended when using AI agents.
* Write a physics-informed test using pytest to validate LLM output against known textbook results.
:::

## The Trust Problem in Scientific AI

Large Language Models (LLMs) are highly capable of writing syntactically correct Python or C++ code. However, they lack intrinsic physics intuition. A model might generate code that runs perfectly without errors but calculates a cross-section, invariant mass, or momentum vector incorrectly.

To prevent this, we should use **Scientific-informed guardrails**. Before asking the AI to write a function, the user must define the constraints using a testing framework like pytest.

## Test-Driven Development (TDD) with LLMs

As an example, below is a common High-Energy Physics (HEP) scenario: calculating the invariant mass of a particle from two daughter four-vectors.

Instead of asking the LLM to write the code first, we write the test first based on a known physics truth.

```python
# test_kinematics.py
import math

# We assume the LLM will write a function called calculate_invariant_mass
# inside a file named kinematics.py
from kinematics import calculate_invariant_mass

def test_invariant_mass_two_photons():
    # Known textbook case: two massless photons moving in opposite directions
    # along the z-axis with E1 = 10 GeV and E2 = 10 GeV.
    # The invariant mass should be exactly 20 GeV.

    # Particle 1 (E, px, py, pz)
    e1, px1, py1, pz1 = 10.0, 0.0, 0.0, 10.0
    # Particle 2 (E, px, py, pz)
    e2, px2, py2, pz2 = 10.0, 0.0, 0.0, -10.0

    mass = calculate_invariant_mass(e1, px1, py1, pz1, e2, px2, py2, pz2)

    # We use math.isclose to account for small floating-point errors
    assert math.isclose(mass, 20.0, rel_tol=1e-3), f"Expected 20.0, but got {mass}"
```

:::{admonition} Exercise: Generating and Validating the Code
:class: tip

1.  Save the code above in a file called test_kinematics.py.
2.  Open your preferred AI assistant (ChatGPT, Claude, or an agentic IDE).
3.  Prompt the AI: "Write a Python function calculate_invariant_mass(e1, px1, py1, pz1, e2, px2, py2, pz2) that calculates the relativistic invariant mass of
    two particles. Return only the code."
4.  Save the AI output in a file called kinematics.py.
5.  Run the test in your terminal by executing pytest test_kinematics.py.

Did the AI get the physics right on the first try? :::

::::{admonition} Solution
:class: dropdown
Often, standard LLMs might use naive Euclidean geometry instead of the Minkowski metric formulas (i.e., they might add the squares instead of subtracting the momentum squared from the energy squared), causing the test to fail.

If the test fails, do not fix the code yourself. Instead, use the test failure as a prompt to create an iterative feedback loop:

"Your code failed this pytest with the error: AssertionError: Expected 20.0, but got 0.0. You likely used standard spatial distance instead of the relativistic invariant mass formula. Please correct the calculation."
::::

:::{admonition} Key Points
:class: important

  - The human is always responsible for the correctness of scientific code.
  - Never deploy AI-generated physics functions without automated unit tests checking edge cases and known limits.
  - Use pytest output as direct feedback to prompt the LLM to fix its own hallucinations.
:::
