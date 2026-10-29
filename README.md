# Statistical Analysis Toolkit

> **Squad Python Specialization — Data Science, AI & Analytics**  
> *A robust, plain-Python statistical analysis and data hygiene toolkit built without relying on built-in modules like `statistics`.*

---

##  Overview & Approach

This project implements a complete custom statistical analysis pipeline from scratch using pure Python. The core philosophy is to build transparent, highly efficient, and modular statistical primitives while delegating verification to industry-standard libraries (`NumPy` and `Pandas`) strictly for validation purposes.

The codebase is organized around an object-oriented architecture via the **`StatisticsAnalyzer`** class:
* **Modular Primitives:** Standalone functions handle core math (`mean`, `median`, `mode`, `variance`, `std_dev`, and `quartiles`).
* **Performance Optimization:** Sorted data and intermediate quartile computations are cached within the `StatisticsAnalyzer` class to prevent redundant $O(N \log N)$ sorting operations on every query call.
* **Data Hygiene & Auditing:** Automated extraction logs rejected rows and invalid data fields cleanly alongside their row indices.

---

##  Data Handling: What Counts as Invalid?

During CSV ingestion via `csv.DictReader`, every data point undergoes rigorous casting checks:
* **Validation Rule:** Each raw cell value is explicitly cast to a floating-point number (`float()`).
* **Failure Criteria:** Any value that raises a `ValueError` is categorized as **invalid/rejected**. This includes:
  * Empty cells or missing values (e.g., `""`).
  * Non-numeric text strings (e.g., `"abc"`, `"n/a"`).
  * Malformed numeric formats containing unexpected characters or typos (e.g., `"-"`, `"12,40O"` where a letter `O` replaces a zero).
* **Logging:** Rejected values are captured alongside their exact CSV row numbers (adjusted for header offset) and reported prior to analysis so analysts can audit source files.

---

##  Methodological Choices & Assumptions

### 1. Sample vs. Population
* **Choice:** **Sample Statistics ($n - 1$ degrees of freedom)** are used by default for variance and standard deviation calculations (`sample=True`).
* **Justification:** The datasets (e.g., agent deposits and transactions) are treated as samples drawn from a broader underlying operational population rather than exhaustive census data.

### 2. Quartile Method
* **Choice:** **Median-of-Halves Method**.
* **Justification:** Quartiles ($Q_1, Q_2, Q_3$) are computed by splitting the sorted dataset around the median. For odd-length datasets, the median element is excluded from both halves; for even-length datasets, the dataset is split evenly down the center.

### 3. Outlier Detection
* **Choice:** **Interquartile Range (IQR) Fence Method** with a default multiplier of $k = 1.5$.
* **Definition:** 
  $$\text{IQR} = Q_3 - Q_1$$
  $$\text{Lower Fence} = Q_1 - 1.5 \times \text{IQR}$$
  $$\text{Upper Fence} = Q_3 + 1.5 \times \text{IQR}$$
  Any data point falling outside these fences is flagged as an outlier.

### 4. Assumptions
* Input files follow a comma-separated values (CSV) format with a structured header row.
* Numeric columns may contain intermittent dirty data that requires exception handling rather than crashing the pipeline.
* Floating-point precision is sufficient for business financial figures (e.g., KES daily deposits).

---

##  How to Run the Report

You can execute the toolkit directly via the provided Jupyter Notebook or by converting/running the core Python script:

1. **Ensure dependencies are installed** (required only for comparison validation):
   ```bash
   pip install numpy pandas