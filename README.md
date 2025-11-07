# kl-project-2025

# Process
Each JSON file in the *SciERC dataset* contains:
- **Documents**
- **Sentences** belonging to those documents
- **Relations** between entities within those sentences

The goal is to generate **questions for each document** so that the LLM can produce **triplets** in the same structured format as defined by the SciERC dataset.  
These triplets follow the structure:

$$[subject:label, relationship, object:label]$$
---
## Entity and Relation Labels

Both entities and relations are constrained to a predefined set of valid labels.

### Entity Labels
- **Method**
- **Task**
- **Dataset**

### Relation Labels
- **Used-For**
- **Part-Of**
- **Compare-With**
- **SubClass-Of**
- **Synonym-Of**
- **Evaluated-With**
- **Benchmark-For**
- **Trained-With**
- **SubTask-Of**
---
## Experimental Procedure

During the experiment, the LLM must be **prompted with clear and specific instructions** to ensure the correct extraction of these triplets for later comparison.

Because the *SciERC dataset* is quite extensive, only a **subset** of the available documents from the different JSON files should be used.  
This makes the experiment computationally feasible while maintaining representative coverage of entity and relation types.
---
## Gold Standard Construction

The **gold standard Knowledge Graph (KG)**, used as the reference for evaluation, must correspond **exactly** to the same set of documents for which the LLM-generated triplets were produced.  

Including any additional documents in the gold standard that were **not** part of the LLM question set would introduce **bias** into the comparison process.
---

# Usage of files

| Stage of Your Methodology                       | Relevant Files                          | How to Use                                                                                       |
| ----------------------------------------------- | --------------------------------------- | ------------------------------------------------------------------------------------------------ |
| Extract the "golden standard" Knowledge Graph   | `train.jsonl` and `dev.jsonl`           | Train or validate a KG extractor that creates graphs from the articles.                          |
| Generate questions for each document            | All (preferably `train.jsonl`)          | Use the article texts (or summaries) to generate QA pairs related to Task/Method/Dataset.        |
| Perform Knowledge Injection (RAG, etc.)	        | `test.jsonl`                            | Use the LLM to answer questions with and without knowledge injection.                            |
| Extract KG from LLM responses                   | `test.jsonl`, `test_ood.jsonl`          | Automatically extract a KG from the generated LLM responses.                                     |
| Evaluate hallucinations (with RefChecker)       | `test.jsonl` and `test_ood.jsonl`       | Compare the "golden" KG (from the dataset) with the LLM's KG, and measure the hallucination rate.|

Extract the “golden standard” Knowledge Graph

Use: train.jsonl and dev.jsonl
- These are the annotated and trusted datasets, ideal for training and validating your KG extraction model.
- You don’t use test or test_ood here because they are meant for final evaluation, not training.

Generate questions for each document

Use: Preferably train.jsonl (or all)
- train contains many diverse examples, perfect for generating question–answer pairs used during training.
- You can include dev and test for more coverage, but the focus here is training the question generation process, not evaluation.

Perform Knowledge Injection (RAG, etc.)

Use: test.jsonl
- This is the main test set, used to evaluate how well the LLM answers questions with and without knowledge injection.
- Avoid using train or dev here to prevent data leakage or bias.

Extract KG from LLM responses

Use: test.jsonl and test_ood.jsonl
- These are your evaluation datasets — test checks performance on familiar domains, while test_ood measures generalization to unseen scientific domains.
- You don’t use train or dev here, since the goal is evaluation only.

Assess hallucinations (RefChecker or similar)

Use: test.jsonl and test_ood.jsonl
- They allow comparing the golden standard KG from the dataset with the KG extracted from the LLM’s answers.
- This way, you can measure both accuracy and robustness of your Knowledge Injection methods.

# Questions (Must be revised, questions must be related with the triplets)
Q1 — (Easy, factual recall)

Question: According to the RCN excerpt, what does “RCN” stand for?
Probes: simple name recall (surface-level hallucination).
Ground-truth: Recurrent Convolutional Network.

Q2 — (Easy, citation detail)

Question: Which dataset(s) are explicitly mentioned as used to evaluate the proposed RCN?
Probes: recall of dataset names (tests exact-phrase memory vs. inventing).
Ground-truth: Kinetics and MultiThumos.

Q3 — (Easy–Medium, method property)

Question: List three desirable properties the authors claim RCN has that address problems of 3D CNNs.
Probes: extraction of enumerated claims (hallucination when model invents additional properties).
Ground-truth: causality (causal outputs / online processing), preserves temporal resolution (frame-level predictions), allows flexible temporal reasoning via recurrence. (Also: uses fewer parameters; benefits from ImageNet initialization — acceptable as extra true claim.)

Q4 — (Medium, comparison)

Question: The text says some works decompose 3D convolutions into separate spatial and temporal convolutions. What name/architecture is given for that decomposition?
Probes: recall of architecture names vs. making up names.
Ground-truth: S3D (separate 2D spatial + 1D temporal convolutions).

Q5 — (Medium, numeric/detail)

Question: The CornerNet excerpt reports an AP on MS COCO. What numeric AP value is given?
Probes: numeric hallucination (models often make up numbers).
Ground-truth: 42.2% AP.

Q6 — (Medium–Hard, provenance & method nuance)

Question: In the RCN text, how do the authors justify using ReLU activations for their recurrent components? Cite the reason given.
Probes: short provenance / justification recall (hallucination if model invents different rationale).
Ground-truth: They use ReLU because of fast convergence and sparsity properties (and cited standard practice in CNNs).

Q7 — (Hard, adversarial robustness)

Question: From the self-attention / adversarial excerpt: name one limitation of the GS-GR adversarial method and the complementary method the authors say produces more natural adversarial examples.
Probes: understanding and recalling critical assessment (hallucination when inventing methods/claims).
Ground-truth: Limitation of GS-GR: often produces unnatural adversarial examples that can change semantics (e.g., replacing a positive word with its antonym). Complementary method: GS-EC (and AS_MIN-GR / AS_MAX-GR discussed) — GS-EC is described as generating more natural, semantically consistent adversarial sentences with similar success rate.

Q8 — (Hard, model-scope / claim-check)

Question: The GxVAEs paper claims to combine two VAEs. What are their roles and how are they connected? (short answer)
Probes: multi-part architecture understanding — hallucination if structure invented or reversed.
Ground-truth: ProfileVAE extracts latent features from gene-expression profiles; those latent features condition MolVAE, which generates hit-like molecules. In short: ProfileVAE → condition → MolVAE.

Q9 — (Very hard, cross-document consistency / counterfactual)

Question: The RCN excerpt states that inflated 2D → 3D networks (I3D) benefit from ImageNet initialization. Does the text claim that RCN shows similar improvements when initialized from ImageNet? Provide the exact claim and indicate whether the text asserts RCN performs better, worse, or comparably to I3D when using ImageNet initialisation.
Probes: fine-grained wording / comparative claim checks (very likely place for hallucination).
Ground-truth: The text asserts that RCN “exhibits similar performance improvement gains when it comes to ImageNet initialisation as those of inflated 3D CNNs (I3D).” It further claims RCN outperforms baseline I3D models overall, while also benefiting from ImageNet initialization.
(So: similar improvement from initialization; overall RCN claimed to perform better than baseline I3D.)

Q10 — (Very hard, challenge for fabricated citation / invented experiment)

Question: Find a direct claim in the provided context that would contradict the following invented statement — “Tran et al. showed that 3D CNNs dramatically and universally outperform 2D CNNs on Sports-1M” — and cite the supporting sentence from the excerpts (paraphrase accepted). What does the true excerpt actually report about Tran et al.’s early 3D CNN results?
Probes: detection of hallucinated counterfactuals and retrieving exact contradictory evidence (exposes fabrication).
Ground-truth (paraphrase): The excerpt states that when Tran et al. first proposed 3D CNNs, their observed performance was “merely comparable to that of 2D CNNs,” e.g., on Sports-1M — directly contradicting the invented claim that they dramatically/outperform 2D CNNs.