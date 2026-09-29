LLM can produce incorrect result due to wrong setup or hallucination. 
This chapter will be focused on how to detect and measure the mistakes and incorrect code (instruction) produced by agent.

#### Hallucination and Uncrertainties

```python
Example
```

#### Code validation.Tool calling
One of the popular ways to validate produced code is to use an external tool for that. 
This is a straightforward way as it doesn't require any test preparation and condition specification.
Static code analysis inspects the code and verifies whether it contains some syntactic mistakes, wrong type usage.

```python
Example
```

In case the developer wants to inspect the structural context and dependencies, then one might use the abstract syntax tree parser.

####  Code validation. LLM as a judge
Another very popular technique to validate the produced code is to use another LLM. 
In this way, the agentic behavior will assemble actor-critic architecture where the first actor is the LLM that produced the code, is verified by another LLM. 
This architecture often calls LLM as a judge, and it can be much more better than other tools as it can provide some insights concerning the explanation concerning the mistakes. 

```python
Example
```
