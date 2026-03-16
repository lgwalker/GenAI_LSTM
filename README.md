# LSTM Java Method Summarizer

This project implements a sequence-to-sequence Long Short-Term Memory (LSTM) model designed to generate natural language summaries (JavaDoc style) for Java methods. By leveraging frozen CodeT5+ embeddings and a dual-layer LSTM architecture, the model learns to map complex source code structures to concise, human-readable descriptions.

---

## Project Overview

The core objective is to automate the generation of documentation for Java methods. The pipeline involves large-scale data collection from GitHub, structural parsing using Abstract Syntax Trees (AST), and a sophisticated training loop with semantic evaluation metrics.


## Data Sources & Pre-processing

### 1. Data Source

- **Source:** Top-starred Java repositories fetched via the GitHub Search API.
- **Filter:** Repositories with `stars > 100`.
- **Scale:** The pipeline targets a dataset of approximately **52,000** method-summary pairs.

### 2. Pre-processing Pipeline

- **AST Parsing:** Uses `javalang` to parse Java source files, ensuring only valid method declarations and their associated documentation are extracted.
- **Cleaning:**
  - Summaries are stripped of HTML tags, `@` annotations, and non-ASCII characters.
  - Only the first sentence of a JavaDoc is retained and converted to lowercase to focus on the primary method intent.
- **Filtering:**
  - Methods must be between 10 and 2,000 characters.
  - Summaries must be between 3 and 20 words.
- **Tokenization:** Code and summaries are tokenized using the `Salesforce/codet5p-220m` tokenizer.
- **Splitting:**

| Split      | Samples |
|------------|---------|
| Training   | 50,000  |
| Validation | 1,000   |
| Test       | 1,000   |


## Installation & Reproduction

### 1. Dependencies

Installed with python 3.9

```bash
pip install -r requirements.txt
```

Ensure the files correlated to [Side model weights](https://drive.google.com/drive/folders/1QdxrYnelt9poi45eLT5xgObihDRb_OtV) are downloaded and stored in the root directory of:
```
./side_model
```

### 2. Reproduction Steps

Follow the execution order below to reproduce results:

1. **Extraction** — Run the data collection module to clone repositories and extract method-summary pairs.
2. **Embeddings** — Generate embeddings using the `get_codet5_embeddings.py` script.
3. **Model Training** — The `LSTMSummarizer` uses a 2-layer LSTM with frozen CodeT5+ embeddings. Training includes early stopping with a patience of **3 epochs** to prevent overfitting.
4. **Evaluation** — After training, the system automatically loads `best_lstm_model.pt` and evaluates the test set using:
   - **BLEU & METEOR** (N-gram overlap)
   - **BERTScore** (Semantic similarity)
   - **SIDE** (Semantic Identity metric)


## Output Locations

All generated artifacts are organized within the `dataset/` directory and the root folder:

| Artifact               | Location                                                          |
|------------------------|-------------------------------------------------------------------|
| Cloned Repositories    | `dataset/java_repos/`                                             |
| Raw Text Splits        | `dataset/summarization/{train,val,test}_{code,summary}.txt`       |
| Tokenized Tensors      | `dataset/summarization/{train,val,test}_{code,summary}.pt`        |
| Embeddings Scripts     | `get_codet5_embeddings.py`                                        |
| Best Model Checkpoint  | `best_lstm_model.pt`                                              |
| Final Predictions      | `assignment2_predictions.json`                                    |

---

