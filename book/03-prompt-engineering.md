# Prompt Engineering

:::{admonition} Overview
:class: note

**Questions**

* What makes a prompt useful for a scientific coding task?
* What context should we provide to a Large Language Model (LLM)?
* How can we make the model's assumptions visible?
* How can we improve a prompt when the first response is incomplete or incorrect?

**Objectives**

* Write clear prompts with relevant context, constraints, and expected output.
* Make assumptions explicit and use examples when they help clarify the task.
* Refine prompts iteratively based on the model’s response.

:::

## What is prompt engineering?

A **prompt** is the instruction or input given to a generative AI system. **Prompt engineering** is the practice of designing and refining that input so that the model is more likely to produce a useful response.

The wording, structure, context, and examples included in a prompt can all affect the result. Prompt engineering is therefore less about finding a set of "magic words" and more about **communicating the task clearly**.

For scientific software, this is especially important because a short request can hide many domain-specific choices. For example, "calculate HT" does not tell the model which jets to use, what cuts to apply, what input structure is available, or whether one value is required per event.

A useful mental model is:

> **Task + Context + Constraints + Expected output (+ Examples when useful)**

A clear prompt communicates the task, relevant context, important constraints, and the expected output. Examples can be added when they help clarify the desired behavior.

This chapter focuses on four practical habits:

1. giving enough context,
2. asking the model to explain assumptions,
3. iterating and refining prompts,
4. applying these ideas to Python and HEP workflows.

## How to write useful prompts

A useful prompt should make the goal easy to identify and should reduce unnecessary ambiguity.

Compare:

> Make a histogram of jet transverse momentum.

with:

> Using Python and Matplotlib, make a histogram of the `pt` field of an Awkward Array called `jets`. Select jets with `pt > 30 GeV` and `abs(eta) < 2.4`. Use 50 bins from 0 to 500 GeV. Label the x-axis `Jet pT [GeV]` and the y-axis `Jets`.

The second prompt tells the model:

* **Task:** make a histogram,
* **Context:** Python, Matplotlib, and an Awkward Array called `jets`,
* **Constraints:** the jet selection and histogram range,
* **Expected output:** a plot with specified binning and labels.

This does not guarantee that the answer is correct, but it reduces the amount of information the model must guess.

:::{admonition} Practical rule
:class: tip

Before sending a prompt, ask yourself:

* Does the model know what I want?
* Does it know what data or code I am working with?
* Does it know the important constraints?
* Does it know what kind of output I expect?

If one of these answers is "no," the prompt may need more information.

:::

## 1. Giving enough context

Context tells the model what it needs to know about the task.

For scientific coding, useful context may include:

* programming language,
* libraries or frameworks,
* input data structure,
* variable definitions,
* units,
* selection requirements,
* relevant code that already exists,
* software or environment constraints,
* desired output format.

For example:

> I am using Python with Awkward Array. Each event contains a variable-length collection of jets with fields `pt` and `eta`. Calculate event-level HT using jets with `pt > 30 GeV` and `abs(eta) < 2.4`. Return one HT value per event.

This is much more useful than:

> Calculate HT.

### Give relevant context, not all available context

More context is not automatically better. A very long prompt containing unrelated code, documentation, or analysis history can make the main task harder to identify.

A better rule is:

> **Provide the information needed to solve the task, not everything you know about the project.**

If the model needs to work with a particular data structure, a small representative example can be more useful than several paragraphs of explanation.

For example:

```python
jets = {
    "pt":  [[120.0, 80.0, 20.0], [55.0]],
    "eta": [[0.4, -1.2, 2.8],    [0.7]]
}
```

Then explain what each outer list represents and what output is expected.

### Specify the audience or level when it matters

The same answer can be written very differently for a beginner, an experienced developer, or someone reviewing production code.

For example:

> Explain the Awkward Array masking step for a learner who knows Python but is new to Awkward Array.

or:

> Give only the minimal implementation; assume the reader is already familiar with Awkward Array.

### Use examples when the desired behavior is hard to describe

Sometimes it is easier to **show** the desired behavior than to describe it. Providing one or more examples is commonly called **one-shot** or **few-shot prompting**.

For example:

> Convert variable names to `snake_case`.
>
> Example:
> `JetPt` → `jet_pt`
>
> Now convert:
> `LeadingMuonEta`

Examples can communicate formatting, style, or expected transformations very efficiently.

:::{admonition} Keep in mind
:class: note

Examples should be representative of the real task. A misleading example can guide the model toward the wrong behavior just as easily as a good example can guide it toward the right one.

:::

## 2. Asking the model to explain assumptions

When information is missing, an LLM may fill in the gaps by making assumptions.

For example:

> Calculate HT for each event.

may leave several questions unanswered:

* Which jets should be included?
* Is there a minimum jet `pt`?
* Is there an `eta` requirement?
* Is HT defined as the scalar sum of jet transverse momenta?
* What does the input structure look like?

Instead of allowing these assumptions to remain hidden, ask the model to make them explicit.

For example:

> Before writing the code, list the assumptions you are making about the input structure and the definition of HT.

or:

> If any part of the task is ambiguous, state your interpretation before implementing it.

This gives the user an opportunity to correct the model **before** relying on the generated code.

### Ask for clarification when guessing would be risky

You can also explicitly instruct the model not to invent missing requirements:

> If you need information that is not provided, ask me before choosing a value or definition.

This is especially useful when the missing information changes the scientific meaning of the result.

:::{admonition} Important
:class: important

An explanation of assumptions is **not a substitute for validation**. It only makes hidden choices easier to inspect.

Testing and validation are discussed elsewhere in this training.

:::

## 3. Iterating and refining prompts

Prompt engineering is usually an iterative process.

The first prompt gives the model an initial description of the task. The response then gives you information about what the model understood correctly and what still needs clarification.

A useful workflow is:

1. **Prompt** — describe the task and relevant context.
2. **Inspect** — read the response carefully.
3. **Identify** — find what is missing, ambiguous, or incorrect.
4. **Refine** — add or correct only the information that matters.
5. **Repeat** — continue until the response is suitable for review and validation.

For example, start with:

> Using Awkward Array, calculate the sum of jet `pt`.

Suppose the model returns:

```python
ht = ak.sum(jets.pt)
```

but you intended one value per event.

A useful follow-up is:

> This sums over the full dataset. I need one value per event. Modify the code so that the reduction is performed over the jet axis while preserving the event structure.

The important point is that you do not need to restart from the beginning. Use the previous response to make the next prompt more precise.

### Keep revisions focused

When working with existing code, focused instructions can prevent unnecessary changes:

> Modify only this function.

> Keep the existing function signature.

> Do not rename existing variables.

> Show only the lines that need to change.

> Briefly explain what changed and why.

For larger tasks, it can also help to split the work into smaller steps rather than asking for an entire analysis pipeline at once.

For example:

1. implement the object selection,
2. inspect the result,
3. calculate the derived quantity,
4. add plotting,
5. add documentation or tests.

This makes each interaction easier to understand and review.

## 4. Examples with Python and HEP workflows

### Example 1: Jet selection

**Vague prompt**

> Write code to select jets.

There is not enough information to know what "select" means.

**Improved prompt**

> Using Python and Awkward Array, select jets with `pt > 30 GeV` and `abs(eta) < 2.4` from an array called `jets`. The array contains one variable-length collection of jets per event and has fields `pt` and `eta`. Preserve the event structure. Briefly explain the masking operation.

The improved prompt specifies the language, library, input structure, selection, expected output, and desired explanation.

### Example 2: Ask for assumptions before calculating HT

> I have an Awkward Array called `jets` with fields `pt` and `eta`. I need to calculate event-level HT. Before writing code, list any assumptions you need to make about the HT definition, object selection, and input structure. If an important definition is missing, ask me rather than choosing one silently.

This helps expose ambiguities before the implementation is generated.

### Example 3: Refining an incorrect reduction

Suppose the model returns:

```python
ht = ak.sum(jets.pt)
```

A focused follow-up could be:

> This produces a single sum over the full dataset. I need one HT value per event. Use the appropriate Awkward Array axis and preserve the event structure.

A revised implementation might be:

```python
ht = ak.sum(jets.pt, axis=1)
```

### Example 4: A complete HEP-style coding prompt

> I am working in Python with Awkward Array. My `jets` array contains a variable number of jets per event with fields `pt` and `eta`.
>
> I want to:
>
> 1. select jets with `pt > 30 GeV` and `abs(eta) < 2.4`,
> 2. calculate event-level HT as the scalar sum of the selected jet `pt`,
> 3. require at least two selected jets,
> 4. return the HT values only for events that pass the selection.
>
> Before writing code, state any assumptions. If an important definition is missing, ask for clarification. Then provide a minimal implementation and briefly explain each Awkward Array operation.

This prompt is relatively detailed, but each detail has a purpose.

## Exercise: Improve a HEP prompt

:::{admonition} Exercise
:class: tip

Consider the following prompt:

> Plot MET.

Rewrite it so that an LLM has enough information to produce a useful Python example for a HEP analysis.

Try to specify:

* the programming language and plotting library,
* the name and structure of the input,
* any event selection,
* the plotting range and number of bins,
* axis labels,
* the expected form of the answer.

If any physics definition is intentionally left unspecified, ask the model to identify it instead of silently assuming it.

:::

::::{admonition} One possible solution
:class: dropdown

> Using Python and Matplotlib, plot the `MET_pt` field from an Awkward Array called `events`. Use only events with at least two jets satisfying `pt > 30 GeV` and `abs(eta) < 2.4`. Plot MET from 0 to 500 GeV using 50 bins. Label the x-axis `MET [GeV]` and the y-axis `Events`. Return the plotting code and a short explanation of the event selection. Before writing the code, state any assumptions about the structure of the `events` object.

There is no single correct version of the prompt. The goal is to reduce ambiguity by clearly communicating the task, context, constraints, and desired output.

::::

## Exercise: Refine rather than restart

:::{admonition} Exercise
:class: tip

You asked an LLM:

> Calculate event-level HT using Awkward Array.

It returned:

```python
ht = ak.sum(jets.pt)
```

Write a **follow-up prompt** that corrects the problem without restating the entire task.

:::

::::{admonition} One possible solution
:class: dropdown

> This reduces over the entire array. I need one HT value per event. Please sum over the jet axis (`axis=1`) while preserving the event structure. Do not change anything else.

This is an example of iterative prompting: use the model's previous answer to communicate exactly what needs to change.

::::

:::{admonition} Key Points
:class: important

* A useful prompt clearly communicates the **task, context, constraints, and expected output**.
* Give enough context to remove important ambiguity, but avoid unrelated information.
* Use representative examples when they communicate the desired behavior more clearly than instructions alone.
* Ask the model to state assumptions or request clarification when missing information could change the result.
* Treat prompting as an iterative process: inspect the response and refine the request.
* In scientific software, better prompting can reduce ambiguity, but it does not replace human review, testing, or validation.

:::

## Further reading

The following resources provide additional introductions and practical guidance on prompt engineering:

* [AWS — What is Prompt Engineering?](https://aws.amazon.com/what-is/prompt-engineering/)
* [IBM — Prompt Engineering Techniques](https://www.ibm.com/think/topics/prompt-engineering-techniques)
* [Oracle — What Is Prompt Engineering?](https://www.oracle.com/artificial-intelligence/prompt-engineering/)
* [Google Cloud — What is Prompt Engineering?](https://cloud.google.com/discover/what-is-prompt-engineering)
* [Stanford HAI — What is Prompt Engineering?](https://hai.stanford.edu/ai-definitions/what-is-prompt-engineering)
