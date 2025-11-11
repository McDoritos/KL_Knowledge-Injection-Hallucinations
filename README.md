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

Five documents from the training dataset were randomly selected and had gold standard KG's generated for them, the questions that will be prompted to the LLM must now representitive enough of this documents in order for the LLM to construct a good KG.

---
## Gold Standard Construction

The **gold standard Knowledge Graph (KG)**, used as the reference for evaluation, must correspond **exactly** to the same set of documents for which the LLM-generated triplets were produced.  

Including any additional documents in the gold standard that were **not** part of the LLM question set would introduce **bias** into the comparison process.

---