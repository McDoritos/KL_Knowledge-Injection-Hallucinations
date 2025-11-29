# Gathering of the Gold Knowledge Graph
The SciERC dataset contains a large collection of annotated scientific papers. However, due to the academic scope and time constraints of the current project, the dataset was significantly reduced.

From the original 106 documents, only 5 were randomly selected for analysis:
- doc_146120936
- doc_201070522
- doc_202888986
- doc_204901567
- doc_208548469

For simplicity, these will be referred to as Doc-1, Doc-2, and so on.
The random selection was performed using the seed 42.

The extraction process produced the following Knowledge Graphs (KGs):
- Doc-1: 80 nodes, 125 edges, 71 chunks
- Doc-2: 91 nodes, 146 edges, 102 chunks
- Doc-3: 53 nodes, 77 edges, 46 chunks
- Doc-4: 49 nodes, 86 edges, 61 chunks
- Doc-5: 70 nodes, 144 edges, 58 chunks

For contextual reference, the KG extracted from the full SciERC dataset contains 5,640 nodes and 12,083 edges.

All extractions were performed using the script `kg_extractor.ipynb`.

---
# Generating a set of questions for each document
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
# RAG procedure
As the purpose of the current project is to access how different methods of knowledge injection affect hallucination rate on the extraction of KG's, the LLM's output.

We first need to create a vector database for the RAG to retrieve relevant information for each of the questions prompted to the LLM. This is done by:
- Retrieving all chunks (phrases) from the content files;
- Embedding each chunk, through a embedding model;
- Storing all embeded chuncks on a vector database.

After this based on the prompt given to the LLM, in this context a question, the embedding model embeds as well the query and a cosine similarity is calculated between this query and every single embedding on the vector database, having this only the top 3 most similar chuncks are retrieved and will support the LLM output.

# LLM prompt
As told before the LLM should be prompted with **prompted with clear and specific instructions** to ensure the correct extraction of these triplets. So the following prompt template was created:

```Text
You are an information extraction system trained to produce 
Knowledge Graph (KG) triplets following the SciERC schema.

========================================
TASK
========================================
Given a research question and a set of retrieved scientific 
document chunks, extract ONLY the factual triplets that satisfy 
all of the following rules:

1. Triplet format:
   [subject:ENTITY_LABEL, RELATION_LABEL, object:ENTITY_LABEL]

2. Allowed ENTITY_LABEL values:
   - Method
   - Task
   - Dataset

3. Allowed RELATION_LABEL values:
   - Used-For
   - Part-Of
   - Compare-With
   - SubClass-Of
   - Synonym-Of
   - Evaluated-With
   - Benchmark-For
   - Trained-With
   - SubTask-Of

4. A triplet must be:
   - Explicitly stated OR unambiguously inferable from the context.
   - Supported ONLY by the retrieved context. 
   - Never invented, guessed, or hallucinated.

5. If the context does NOT support any triplets:
   - Return an empty list: []

6. Do NOT:
   - Add new entities not mentioned in the retrieved context.
   - Use labels that do not follow the SciERC schema.
   - Produce explanations—ONLY the triplets list.

========================================
QUESTION
{QUESTION}

========================================
RETRIEVED CONTEXT
{RETRIEVED_CONTEXT}

========================================
OUTPUT FORMAT
Return ONLY a JSON object with the following field:
{
  "triplets": [
     "[subject:LABEL, RELATION, object:LABEL]",
     ...
  ]
}

If no valid triplets exist, output:
{ "triplets": [] }

```

# Extracting KG
From the LLM’s output, a Knowledge Graph (KG) must be extracted for each document so that they can be evaluated individually. Since the prompting strategy enforces the LLM to output triplets following the same structure as the SciERC dataset, the extracted KGs can be directly compared to the gold-standard KGs using the RefChecker framework.

# Assessing Hallucinations
Once all KGs have been extracted, the final step is to assess hallucinations. As previously discussed, this evaluation is performed using the RefChecker approach, which consists of the following components:
- Claim Extraction (E) — This step is already fulfilled, as the LLM is explicitly instructed to output triplets, which form the basis for KG construction.
- Hallucination Verification (C) — Using the gold-standard KG, each LLM-generated triplet is compared against the reference triplets. Based on this comparison, each claim is classified as supported, contradicted, or not verifiable.
- Aggregation (A) — Finally, hallucination metrics are computed for each document, aggregating the verification results to quantify the model’s performance.

