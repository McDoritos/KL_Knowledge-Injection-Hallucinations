# kl-project-2025

| Stage of Your Methodology                       | Relevant Files                          | How to Use                                                                                       |
| ----------------------------------------------- | --------------------------------------- | ------------------------------------------------------------------------------------------------ |
| Extract the "golden standard" Knowledge Graph   | `train.jsonl` and `dev.jsonl`           | Train or validate a KG extractor that creates graphs from the articles.                          |
| Generate questions for each document            | All (preferably `train.jsonl`)          | Use the article texts (or summaries) to generate QA pairs related to Task/Method/Dataset.        |
| Perform Knowledge Injection (RAG, etc.)	        | `test.jsonl`                            | Use the LLM to answer questions with and without knowledge injection.                            |
| Extract KG from LLM responses                   | `test.jsonl`, `test_ood.jsonl`          | Automatically extract a KG from the generated LLM responses.                                     |
| Evaluate hallucinations (with RefChecker)       | `test.jsonl` and `test_ood.jsonl`       | Compare the "golden" KG (from the dataset) with the LLM's KG, and measure the hallucination rate.|
