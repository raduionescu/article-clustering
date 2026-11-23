# Article Clustering

Hierarchical clustering of scientific articles using **BERT embeddings** or **TF-IDF features**, with a focus on a bibliography of glioblastoma (GBM) research.

The project reads article titles and abstracts from an Excel workbook, converts each article into a numerical feature vector, applies Ward hierarchical clustering, and exports a labelled dendrogram as a PDF.

## Repository Contents

```text
article-clustering/
├── GBM_Bibliography_Abstracts.xlsx
├── ward_clustering_on_bert_embeddings_or_tfidf_v3.py
└── README.md
```

- `GBM_Bibliography_Abstracts.xlsx` contains the article bibliography.
- `ward_clustering_on_bert_embeddings_or_tfidf_v3.py` contains the feature extraction, clustering, and visualization pipeline.

## Dataset

The script expects the Excel workbook to contain a sheet named `Bibliography` with the following columns:

| Column | Description |
|---|---|
| `ID` | Article identifier, typically the author name or citation label |
| `Title` | Article title |
| `Abstract` | Article abstract |

Each article is represented by concatenating its title and abstract:

```python
text = row['Title'] + ' ' + row['Abstract']
```

The included workbook contains 149 bibliography records plus the header row.

## Method

The pipeline supports two document representations.

### BERT embeddings

The script loads `bert-base-uncased` from Hugging Face Transformers and tokenizes each concatenated title and abstract. Inputs longer than 512 tokens are reported and truncated to the model limit.

For each article, the last hidden layer is mean-pooled across tokens to produce a dense feature vector:

```python
last_hidden_states = outputs.hidden_states[-1]
feature_vector = torch.mean(last_hidden_states[0], dim=0)
```

### TF-IDF features

The script also provides a TF-IDF representation using scikit-learn:

```python
tfidf_vectorizer = TfidfVectorizer(stop_words='english')
tfidf = tfidf_vectorizer.fit_transform(df['text'])
```

In the current script, the BERT vectors are computed first and then replaced by the TF-IDF vectors before clustering. Therefore, the default final clustering output uses **TF-IDF**. To cluster the BERT representations instead, remove or comment out this line:

```python
feature_vectors, IDs = extract_tfidf(df)
```

## Clustering

The script performs agglomerative hierarchical clustering with Ward linkage and Euclidean distance:

```python
linkage_matrix = linkage(feature_vectors, method='ward')
```

It also computes inconsistency statistics and prints the mean maximum inconsistency coefficient.

## Requirements

- Python 3.9 or newer
- pandas
- openpyxl
- NumPy
- PyTorch
- Transformers
- scikit-learn
- SciPy
- Matplotlib

Install the dependencies with:

```bash
python -m pip install pandas openpyxl numpy torch transformers scikit-learn scipy matplotlib
```

> The first BERT run downloads the `bert-base-uncased` tokenizer and model weights from Hugging Face. An internet connection and sufficient disk space are required for that initial download.

## Usage

### 1. Clone the repository

```bash
git clone https://github.com/raduionescu/article-clustering.git
cd article-clustering
```

### 2. Create and activate a virtual environment

Linux or macOS:

```bash
python -m venv .venv
source .venv/bin/activate
```

Windows PowerShell:

```powershell
python -m venv .venv
.venv\Scripts\Activate.ps1
```

### 3. Install the dependencies

```bash
python -m pip install pandas openpyxl numpy torch transformers scikit-learn scipy matplotlib
```

### 4. Add the source workbook

Place `GBM_Bibliography_Abstracts.xlsx` in the repository root, next to the Python script. The workbook must contain the `Bibliography` sheet and the `ID`, `Title`, and `Abstract` columns.

### 5. Run the clustering pipeline

```bash
python ward_clustering_on_bert_embeddings_or_tfidf_v3.py
```

The script prints progress information while extracting BERT features, reports articles that exceed 512 tokens, prints the number of records and the mean maximum inconsistency coefficient, and then saves the dendrogram.

## Output

The PDF contains a hierarchical clustering dendrogram labelled with the article IDs from the workbook.

## License

This repository is licensed under the [CC-BY 4.0](https://creativecommons.org/licenses/by/4.0/deed.en) 
