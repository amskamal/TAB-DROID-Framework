# 🤖 TAB-DROID — Android Malware Detection Framework

A machine learning-based framework for **Android malware detection and analysis**, built using **Jupyter Notebooks** and trained on well-known Android malware datasets including **Malgenome** and **TUANDROMD**. The project covers the full ML pipeline from raw data preparation and cleaning through to model training and evaluation.

---

## 🛠️ Tech Stack

| Layer | Technology |
|---|---|
| Language | Python |
| Environment | Jupyter Notebook |
| Datasets | Malgenome, TUANDROMD |
| Core Libraries | scikit-learn, skfeature-chappers, nanopq, TabPFN |

---

## 📦 Libraries & Dependencies

### Installation

```bash
pip install skfeature-chappers
pip install nanopq
pip install tabpfn
```

The following libraries are used across the notebooks in this project:

---

### 🔵 `scikit-learn` — Machine Learning Framework

```python```

**scikit-learn** is the core machine learning library used throughout the project for training, evaluating, and comparing classification models.

| Module | Class / Function | Purpose |
|---|---|---|
| `sklearn.model_selection` | `train_test_split`, `KFold` | Splits data into training and test sets, and performs k-fold cross-validation |
| `sklearn.metrics` | `accuracy_score`, `classification_report`, `confusion_matrix` | Evaluates model performance using accuracy, precision, recall, F1-score, and confusion matrix |
| `sklearn.preprocessing` | `LabelEncoder`, `StandardScaler` | Encodes class labels and scales features to a standard range |

---

### 🟢 `skfeature-chappers` — Feature Selection Library

```python
from skfeature.function.information_theoretical_based import CMIM, JMI
```

**skfeature-chappers** (version `1.1.0`) It provides a wide range of feature selection algorithms, particularly those based on **information theory**.

In this project, two specific algorithms from the `information_theoretical_based` module are used:

#### `CMIM` — Conditional Mutual Information Maximization

```python
from skfeature.function.information_theoretical_based import CMIM
idx, _, _ = CMIM.cmim(X_train, y_train, n_selected_features=num_features)
```

- **What it does:** Selects the most informative features by maximizing the **conditional mutual information** between each feature and the class label, while minimizing redundancy between already-selected features
- **Why it's used:** Android apps have hundreds of permission and API call features — many of which are redundant. CMIM identifies the smallest subset of features that carry the most unique information about whether an app is malware or benign
- **Reference:** Brown, Gavin et al. *"Conditional Likelihood Maximisation: A Unifying Framework for Information Theoretic Feature Selection."* JMLR 2012

#### `JMI` — Joint Mutual Information

```python
from skfeature.function.information_theoretical_based import JMI
idx, _, _ = JMI.jmi(X_train, y_train, n_selected_features=num_features)
```

- **What it does:** Selects features by maximizing the **joint mutual information** between a pair of features and the class label — it considers the combined interaction of features, not just individual relevance
- **Why it's used:** Some Android malware behaviors are only detectable when multiple features (e.g. a specific permission combined with a specific API call) are observed together. JMI captures these joint dependencies that single-feature methods miss
- **Reference:** Brown, Gavin et al. *"Conditional Likelihood Maximisation: A Unifying Framework for Information Theoretic Feature Selection."* JMLR 2012

**CMIM vs JMI — Key Difference:**

| | CMIM | JMI |
|---|---|---|
| Focus | Minimizes redundancy between features | Maximizes joint relevance of feature pairs |
| Best for | Removing duplicate/correlated features | Capturing feature interactions |
| Complexity | Lower | Slightly higher |

---

### 🟡 `nanopq` — Product Quantization for Nearest Neighbor Search

```python
import nanopq
```

**nanopq** is a pure Python implementation of **Product Quantization (PQ)** and **Optimized Product Quantization (OPQ)** for memory-efficient approximate nearest neighbor search.

- **What it does:** Compresses high-dimensional feature vectors by dividing them into smaller sub-vectors, quantizing each sub-space independently using k-means clustering, and representing each vector as a compact code of integer indices
- **Why it's used in TAB-DROID:** Android malware datasets (especially TUANDROMD with 241 features) produce large, high-dimensional feature vectors. nanopq compresses these vectors so that similarity search and nearest-neighbor comparisons can be performed efficiently without loading all raw vectors into memory and to maintain the data of usered secure by changing its format
- **Key components used:**

```python
pq = nanopq.PQ(M=8)       # Create PQ with 8 sub-spaces
pq.fit(X_train)            # Train codewords via k-means
X_code = pq.encode(X)     # Encode vectors into compact PQ codes
dists = pq.dtable(query).adist(X_code)  # Compute approximate distances
```

| Parameter | Description |
|---|---|
| `M` | Number of sub-spaces the feature vector is split into |
| `Ks` | Number of codewords per sub-space (default: 256 = 8-bit) |
| `metric` | Distance metric: `'l2'` (Euclidean) or `'dot'` (dot product) |

- **Reference:** Jégou et al., *"Product Quantization for Nearest Neighbor Search"*, IEEE TPAMI 2011

---

### 🟣 `TabPFN` — Tabular Prior-Data Fitted Network

```python
from tabpfn import TabPFNClassifier
```

**TabPFN** is a transformer-based tabular classifier, pre-trained on millions of synthetically generated tabular datasets, that performs in-context learning — it takes the training set directly as input at inference time and produces predictions in a single forward pass, with no per-dataset training loop or hyperparameter tuning required.

- **What it does:** Acts as a tabular classifier that approximates Bayesian inference over a wide space of structured-data problems, treating the training data as context rather than fitting model weights to it
- **Why it's used in TAB-DROID:** The Malgenome and TUANDROMD datasets are inherently tabular (permissions, API calls, and intents as columns), so TabPFN serves as a fast, strong-baseline tabular classifier to compare against the project's proposed model — particularly effective on smaller or feature-reduced datasets after `CMIM`/`JMI` selection
- **Key components used:**

```python
from tabpfn import TabPFNClassifier

clf = TabPFNClassifier()
clf.fit(X_train, y_train)
y_pred = clf.predict(X_test)
```

- **Reference:** Hollmann, N. et al. *"TabPFN: A Transformer That Solves Small Tabular Classification Problems in a Second."* ICLR 2023

---

## 📁 Project Structure

```
TAB-DROID/
├── Data-preparation-&-cleaning/    # Data preprocessing, feature extraction & cleaning
├── Malgenome/                      # Notebooks for the Malgenome Android malware dataset
└── TUANDROMD/                      # Notebooks for the TUANDROMD Android malware dataset
```

---

## 📂 Folder Details

### 🧹 `Data-preparation-&-cleaning/`
This folder contains all data preprocessing and feature engineering notebooks — the foundation of the entire pipeline. Before any model can be trained, the raw Android application data must be cleaned, normalized, and transformed into usable feature vectors.

Key tasks covered:
- **Loading raw datasets** from Malgenome and TUANDROMD sources
- **Handling missing values** — identifying and filling or dropping null/NaN entries
- **Removing duplicates** — ensuring no repeated samples skew the model
- **Label encoding** — converting categorical labels (malware / benign) into numeric format
- **Normalization & scaling** using `StandardScaler` from scikit-learn
- **Train/test splitting** — dividing data into training, validation, and test sets
- **EDA** — visualizing class distributions and feature correlations

> ⚠️ Run the notebooks in this folder **first** before running any model notebooks.

---

### 🦠 `Malgenome/`
Notebooks that train and evaluate ML models using the **Android Malware Genome Project (Malgenome)** dataset.

**Dataset facts:**
- 1,260 malware samples collected between 2010–2011
- Covers 49 different malware families (e.g. DroidKungFu, Geinimi, AnserverBot)
- Features extracted from Android APKs: permissions, API calls, and intents

**What the notebooks do:**
- Apply feature selection via `CMIM` / `JMI` to reduce dimensionality
- Train the proposed model on the selected features, with `TabPFN` used as a tabular classification baseline
- Use `nanopq` for efficient vector compression and similarity search
- Evaluate with accuracy, F1-score, confusion matrix, and ROC/AUC

---

### 🦠 `TUANDROMD/`
Notebooks that train and evaluate ML models using the **Tezpur University Android Malware Dataset (TUANDROMD)**.

**Dataset facts:**
- 4,465 instances with 241 attributes each
- Binary classification: malware vs. goodware
- Published in 2020 by Borah, Bhattacharyya & Kalita

**What the notebooks do:**
- Handle the higher dimensionality (241 features) using `CMIM` / `JMI` for feature reduction
- Apply `nanopq` to compress the 241-dimensional vectors for efficient search
- Train and evaluate the proposed model on the reduced feature set, with `TabPFN` used as a tabular classification baseline
- Compare results with Malgenome experiments

---

## ⚙️ Prerequisites

```bash
pip install scikit-learn pandas numpy matplotlib seaborn jupyter
pip install skfeature-chappers
pip install nanopq
pip install tabpfn

```

---

## 🚀 Getting Started

### 1. Clone the Repository

```bash
git clone https://github.com/amskamal/TAB-DROID.git
cd TAB-DROID
```

### 2. Install Dependencies

```bash
pip install scikit-learn pandas numpy matplotlib seaborn skfeature-chappers nanopq tabpfn jupyter
```

### 3. Launch Jupyter Notebook

```bash
jupyter notebook
```

### 4. Run Notebooks in Order

```
Step 1 → Data-preparation-&-cleaning/   ← always run first
Step 2 → Malgenome/                     ← train & evaluate on Malgenome
Step 3 → TUANDROMD/                     ← train & evaluate on TUANDROMD
```

---

## 🤝 Contributing

1. Fork the repository
2. Create a new branch: `git checkout -b feature/your-feature-name`
3. Commit your changes: `git commit -m "Add your feature"`
4. Push to the branch: `git push origin feature/your-feature-name`
5. Open a Pull Request

---

## 📄 License

This project is open source and available under the [MIT License](LICENSE).

---
## Citation

If you use this framework in your research, please cite:

```bibtex

Ahmed M. Saeed,Sameh A. Salem,Shahira M. Habashy,Hadeer A. Hassan, "TAB-DROID: A FRAMEWORK FOR ANDROID MALWARE DETECTION USING THE TABPFN CLASSIFIER", Jordanian Journal of Computers and Information Technology (JJCIT) ,Volume 11,  Number 04, pp. 448 - 465,  December 2025, doi: 10.5455/jjcit.71-1751720957.
```

## 👤 Author

**amskamal** — [GitHub Profile](https://github.com/amskamal)
