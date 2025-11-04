# kl-project-2025

| Etapa da tua metodologia                        | Ficheiros relevantes                    | Como usar                                                                                        |
| ----------------------------------------------- | --------------------------------------- | ------------------------------------------------------------------------------------------------ |
| ① Extrair o *Knowledge Graph* “golden standard” | `train.jsonl` e `dev.jsonl`             | Treina ou valida um extrator KG que cria grafos a partir dos artigos.                            |
| ② Gerar perguntas para cada documento           | Todos (preferencialmente `train.jsonl`) | Usa o texto dos artigos (ou resumos) para gerar *QA pairs* relacionados com Task/Method/Dataset. |
| ③ Fazer *Knowledge Injection* (RAG, etc.)       | `test.jsonl`                            | Usa o LLM para responder às perguntas **com e sem** injeção de conhecimento.                     |
| ④ Extrair KG das respostas do LLM               | `test.jsonl`, `test_ood.jsonl`          | Extrai automaticamente um KG das respostas geradas pelo LLM.                                     |
| ⑤ Avaliar alucinações (com RefChecker)          | `test.jsonl` e `test_ood.jsonl`         | Compara o KG “golden” (do dataset) com o KG do LLM, e mede o grau de alucinação.                 |
