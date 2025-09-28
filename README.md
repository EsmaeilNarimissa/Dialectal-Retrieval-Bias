# Dialectal Bias in Retrieval-Augmented Generation (RAG) Systems

This repository contains the code and documentation for a Proof-of-Concept (PoC) study investigating dialectal bias in the retrieval component of RAG systems. The primary objective is to quantitatively assess whether retrieval performance differs when querying a corpus with Standard American English (SAE) versus synthetically generated African American Vernacular English (AAVE).

## Project Workflow

This project is structured as a two-phase pipeline, with each phase corresponding to a Jupyter notebook:

1.  **Phase 1: Dataset Generation (**`aave_dataset_generation.ipynb`**)**
    -   This notebook creates a parallel dataset of question pairs. It sources questions from the SQuAD 1.1 dataset, filters them for suitability, and then uses a large language model (GPT-4) with carefully designed prompts to generate synthetic AAVE paraphrases for each SAE question. The output is a JSON file containing paired queries, each aligned to a gold-standard context document via its `squad_idx`.
2.  **Phase 2: Retrieval Bias Evaluation (**`rag_bias_poc-2.ipynb`**)**
    -   This notebook ingests the dataset from Phase 1 to run a controlled retrieval experiment. It compares the performance of a sparse retriever (BM25) and a dense retriever (OpenAI's `text-embedding-3-small` + ChromaDB) on both the SAE and AAVE queries. The analysis is multi-faceted, examining performance at different retrieval depths and using multiple metrics to test for bias.

## Methodology

### Phase 1: Paired Dataset Generation

The dataset generation process ensures a controlled and aligned set of queries for the bias evaluation.

1.  **Data Sourcing and Filtering:** Questions and contexts are sourced from the SQuAD 1.1 `train` split. Questions are filtered to select for fact-seeking inquiries (e.g., starting with "Wh-") and to exclude unanswerable or overly complex examples.
2.  **Synthetic AAVE Generation:** A large language model (GPT-4) is prompted with few-shot examples to paraphrase the filtered SAE questions into AAVE. The prompt engineering is designed to encourage dialectal shifts while preserving the original question's core intent.
3.  **Quality Assurance:** A post-generation validation step is performed to ensure that the original answer to the SAE question remains a valid answer for the generated AAVE question, maintaining the integrity of the evaluation pair.
4.  **Output:** The final output is a JSON file where each entry contains an `id`, the original `sae_query`, the synthetic `aave_query`, and the `squad_idx` linking it to its source context.

### Phase 2: Retrieval Bias Evaluation

The evaluation pipeline is designed to rigorously test for performance disparities between the dialects.

1.  **Corpus Preparation:** The SQuAD `train` contexts are loaded, normalized, and deduplicated to create a clean document corpus where each unique context is assigned a stable `doc_id`.
2.  **Multi-Faceted Metrics:** The evaluation goes beyond simple hit rates to provide a comprehensive picture of performance:
    -   **Recall@k:** Measures whether the gold document is present in the top `k` results, tested at `k=5, 10, 20`.
    -   **Rank-of-Gold:** Records the specific rank (position) of the gold document when it is found.
    -   **Hybrid Recall:** Calculates the recall ceiling by combining the results from both BM25 and dense retrievers.
3.  **Statistical Significance Testing:** Paired statistical tests are used to determine if observed differences are meaningful:
    -   **McNemar's Test:** Used for paired binary outcomes (hit/miss) to assess the significance of differences in Recall@k and Hybrid Recall.
    -   **Wilcoxon Signed-Rank Test:** Used to compare the distributions of the Rank-of-Gold metric for SAE vs. AAVE queries.

### How to Run (in Google Colab)

1.  **Set Up API Key:**
    -   In both Colab notebooks, use the **Secrets** tab (key icon on the left-hand sidebar) to add your OpenAI API key. The secret should be named `OPENAI_API_KEY`.
2.  **Execute Phase 1: Generate the Dataset**
    -   Open `aave_dataset_generation.ipynb`.
    -   Run all the cells. The necessary Python packages will be installed automatically by the first code cell.
    -   After execution, a timestamped `.json` file (e.g., `aave_poc_dataset_20250928-123456.json`) will be generated and saved to your Colab session storage.
3.  **Execute Phase 2: Run the Bias Evaluation**
    -   Open `rag_bias_poc-2.ipynb`.
    -   In the Colab file browser (folder icon on the left), find the `.json` file created in Phase 1.
    -   Right-click on the file and select **"Copy path"**.
    -   Paste this copied path into the `aave_dataset_path` variable in the first code cell of the notebook.
    -   Run all the cells. The notebook will install its own dependencies and then use the dataset to perform the full retrieval analysis.

## Key Findings and Conclusion

Across a multi-faceted evaluation, this PoC found **no statistically significant evidence of retrieval bias** against the synthetically generated AAVE queries for either the BM25 or the `text-embedding-3-small` retrieval system.

This conclusion is supported by three key findings:

1.  **Consistent Recall@k:** Performance parity was maintained across all tested retrieval depths (`k=5, 10, 20`), with McNemar's tests confirming the lack of a significant difference.
2.  **No Rank Skew:** The median rank of the correct document was 1.00 for both dialects across both systems, and Wilcoxon tests showed no significant difference in rank distributions.
3.  **Identical Hybrid Ceiling:** The maximum achievable recall, by combining both retrievers, was 99.0% for both SAE and AAVE, with a McNemar's test showing perfect performance agreement.

### Limitations

The primary limitation of this study is its reliance on **synthetically generated AAVE**. The paraphrasing LLM may not fully capture the lexical and syntactic richness of naturally occurring AAVE. The null result suggests that modern retrieval systems are robust to these specific, structured transformations, but this finding may not generalize to all forms of dialectal variation.

Future work should aim to validate these findings using a larger, more diverse dataset that includes naturally occurring AAVE.
