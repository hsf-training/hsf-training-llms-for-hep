# LLM output validation
LLMs can produce incorrect results due to wrong setup or hallucination.
This chapter will focus on how to detect and measure the mistakes and incorrect code (instructions) produced by the agent.

## Hallucination and Uncertainties
Many LLMs, especially small models, don't know the information about the high-energy physics domain well, old codes, or algorithms. This forces them to make mistakes and introduce factual incorrectness. This is possible to catch by measuring the uncertainty level.
For example, semantic entropy, one of the possible uncertainty quantification methods, can be calculated using the algorithm below:

###### Algorithm: Compute Semantic Entropy

**Inputs:**
- $x$: Prompt text
- $\mathcal{M}$: Generation model (LLM)
- $\text{NLI}$: Bidirectional entailment classifier
- $N$: Number of sampled generations
- $T$: Sampling temperature ($T > 0$)

**Output:**
- $\text{SE}(x)$: Semantic Entropy score (scalar)

---

#### Step 1: Sample Generations
1. Initialize sample set $S \leftarrow \emptyset$.
2. **For** $i = 1$ **to** $N$ **do**:
   a. Sample sequence $s_i \sim \mathcal{M}(\cdot \mid x, T)$.
   b. Compute joint log-likelihood:
      $$\log P(s_i \mid x) = \sum_{t} \log P(\text{token}_t \mid x, \text{token}_{<t})$$
   c. Add to sample set: $S \leftarrow S \cup \{(s_i, \log P(s_i \mid x))\}$.

---

#### Step 2: Construct Semantic Equivalence Clusters
1. Initialize empty set of clusters $C \leftarrow \emptyset$.
2. **For** each sequence $s_i \in S$ **do**:
   a. $\text{Matched} \leftarrow \text{False}$
   b. **For** each cluster $c_k \in C$ with representative $s_{\text{rep}}$ **do**:
      - **If** $\text{NLI}(s_i \to s_{\text{rep}}) = \text{ENTAILMENT}$ **and** $\text{NLI}(s_{\text{rep}} \to s_i) = \text{ENTAILMENT}$ **then**:
        - $c_k \leftarrow c_k \cup \{s_i\}$
        - $\text{Matched} \leftarrow \text{True}$
        - **break**
   c. **If** $\text{Matched} = \text{False}$ **then**:
      - Create new cluster $c_{\text{new}} \leftarrow \{s_i\}$
      - $C \leftarrow C \cup \{c_{\text{new}}\}$

---

#### Step 3: Estimate Cluster Probabilities
1. Compute denominator $Z$:
   $$Z = \sum_{i=1}^N \exp(\log P(s_i \mid x))$$
2. **For** each cluster $c_k \in C$ **do**:
   $$P(c_k) = \frac{1}{Z} \sum_{s_i \in c_k} \exp(\log P(s_i \mid x))$$

---

#### Step 4: Calculate Semantic Entropy
1. Initialize $\text{SE}(x) \leftarrow 0$.
2. **For** each cluster $c_k \in C$ **do**:
   - **If** $P(c_k) > 0$ **then**:
     $$\text{SE}(x) \leftarrow \text{SE}(x) - P(c_k) \log P(c_k)$$
3. **Return** $\text{SE}(x)$

```python
Example
```

## Code validation.Tool calling
One of the popular ways to validate produced code is to use an external tool for that.
This is a straightforward way as it doesn't require any test preparation and condition specification.
Static code analysis inspects the code and verifies whether it contains some syntactic mistakes or incorrect type usage.

```python
code_produced_by_llm = '......'
#Validation step:
import io
from pyflakes import api, reporter

stdout_buf = io.StringIO()
stderr_buf = io.StringIO()
rep = reporter.Reporter(stdout_buf, stderr_buf)

warnings = api.check(code_string, filename="<string>", reporter=rep)

stdout_output = stdout_buf.getvalue()
stderr_output = stderr_buf.getvalue()
```

In case the developer wants to inspect the structural context and dependencies, then one might use the abstract syntax tree parser.

##  Code validation. LLM as a judge
Another very popular technique to validate the produced code is to use another LLM.
In this way, the agentic behavior will assemble actor-critic architecture where the first actor is the LLM that produced the code, which is verified by another LLM.
This architecture often calls LLM a judge, and it can be much better than other tools as it can provide some insights into the mistakes.

```python
Example
```
LLM as a judge can usually intoduce the mistakes, even in the flow that produces correct result, so the work will be affected. One of the way to mitigate such undesirable effect is to use the guardrail or estimate the hallucination rate (uncertainty) and add the logical branch based on this value (if-else statement)
