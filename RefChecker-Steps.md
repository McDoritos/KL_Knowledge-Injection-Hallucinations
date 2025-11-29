# RefChecker Functioning

In this work, RefChecker is applied as the final evaluation component of the pipeline designed to measure hallucinations in LLM responses after different Knowledge Injection (KI) strategies are used. The framework operates in three main stages and is compatible with several environments, such as:
- QA without context (Open QA);
- RAG with noisy retrieved context;
- QA with reliable context.

Since our methodology relies on evaluating the LLM’s generated Knowledge Graph (KG) against a gold standard KG, our setting fits the QA with reliable context scenario.

RefChecker consists of the following three core stages:
- Claim Extractor (E);
- Hallucination Verifier (C);
- Aggregator (A).

## First Stage: Claim Extractor

The Claim Extractor decomposes the LLM’s response into knowledge triplets.
In the context of this project, this extraction step is already inherent to the pipeline because the LLM is explicitly prompted to produce a Knowledge Graph from the scientific document. Therefore, each extracted entity–relation–entity triple becomes an individual factual claim to be validated.

This fine-grained triplet representation is crucial because it enables the detection of localized hallucinations within a single sentence or relation, even when parts of the answer may be correct.

## Second Stage: Hallucination Verifier

In this stage, each triplet obtained from the LLM is compared against the gold standard Knowledge Graph corresponding to the same source document (from the SciERC dataset or extracted automatically).

Given that our evaluation compares the LLM output with an authoritative reference KG, we operate under the QA with reliable context environment of RefChecker.

Accordingly, RefChecker receives:
- Source document: the scientific paper from which the gold standard KG was built;
- Question: the document-specific question used to query the LLM;
- Answer: the set of triplets produced by the LLM.

RefChecker assigns each triplet to one of the following categories:
- Supported — the triplet matches the information in the gold standard KG;
- Contradicted — the triplet conflicts with the gold standard;
- Not verifiable — the triplet cannot be confirmed using the available information.

This stage forms the core hallucination detection mechanism, directly quantifying the factual alignment of the LLM’s generated knowledge.

## Third Stage: Aggregator

The Aggregator consolidates the verification results into interpretable evaluation metrics. This module computes:
- the percentage of correct (supported) triplets,
- the percentage of hallucinated triplets (contradicted + not verifiable),
- the absolute number of hallucinations per response.

These metrics allow for a quantitative comparison of hallucination rates across different Knowledge Injection techniques, enabling us to answer the research questions regarding:
- whether different KI methods produce significantly different hallucination levels, and;
- whether hallucination quantification can serve as a reliable benchmarking tool in scientific-domain LLM evaluation.