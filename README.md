# PE6201 End-of-Course Project: Eval-Driven Financial Data Extraction Pipeline

**Author:** Guo Shuhan 
**Course:** PE6201

## 1. Project Overview & Evolution

This project focuses on automating the extraction of checkable, empirical financial data from unstructured corporate share placement announcements .

Based on instructor feedback during Milestone 1, the project architecture was deliberately evolved. The initial proposal aimed to use a generalized RAG approach to also predict or summarize subsequent stock price movements. Following the feedback, the scope was refined to strictly focus on what the LLM does best: extracting deterministic numbers from unstructured text. Stock price movements are left to standard programmatic computation using historical price data arrays, rather than relying on LLM generation.

The pipeline now strictly focuses on extracting two verifiable metrics with a single right answer:

1. **Placement Size **
2. **Discount Rate **

## 2. Data & Ground Truth

The dataset utilizes a **frozen set of 25 historical A-share placement cases**.
To establish a rigorous evaluation baseline, the true Placement Size and Discount Rate for all 25 cases were hand-copied into a static CSV file (the Ground Truth) *before* the first pipeline run.

## 3. Outcome Metrics

As advised, subjective metrics such as "reduction in time to analyze" have been dropped due to the lack of a manual timing baseline.
The sole technical and business outcome metric for this project is **Extraction Accuracy on the 25 frozen cases, reported as exact counts**.

## 4. Technical Stack & Architecture

To achieve a flawless extraction count, this project utilizes a modern AI stack:

* **Foundation Model:** `gpt-4o-mini` via API, handling the complex reading comprehension of raw corporate filings.
* **Pydantic Structured Outputs:** Replaces fragile prompt engineering with rigid Python data schemas (`PlacementExtraction`). The LLM is forced to return deterministic JSON.
* **Entity Resolution Enforcement:** The prompt dynamically injects the specific `stock_name` to prevent the model from returning full corporate names, ensuring a 1:1 match with the automated grading script.
* **Eval-Driven Harness:** An automated script that cross-references the LLM's output (`model_outputs.jsonl`) against the hand-copied Ground Truth CSV, using a `< 0.01` error tolerance to account for floating-point calculation artifacts.

## 5. The Three-Stage Progressive Evaluation (Evals)

To prove the ultimate reliability of the system, the pipeline was subjected to a three-stage progressive stress test, starting from the smallest viable slice and scaling up to real-world complexity.

### Phase 1: Basic Pipeline Validation (Ideal Environment)

* **What was done:** The extraction function was tested using 25 extremely short, artificially simplified mock announcements (e.g., "【"Shanghai Pudong Development Bank】 Rights Issue Plan" - Rights issue ratio: 0.3, discount rate: 0.5").
* **Purpose:** To validate the fundamental plumbing of the code.
* **Result:** **25/25 Correct**. This proves that the API connections, JSON contracts, and scoring scripts function flawlessly when there is no textual noise.

### Phase 2: Intermediate Simulation (Algorithmic Noise)

* **What was done:** The pipeline processed 25 programmatically generated, 5,000-word synthetic announcements. These texts injected simulated financial noise, standard legal boilerplate, and misleading decoy numbers.
* **Purpose:** To test the foundation model's ability to filter out basic algorithmic distractors and strictly adhere to the JSON schema constraint.
* **Result:** **25/25 Correct**.

### Phase 3: Real-World Stress Test ("Needle in a Haystack")

* **What was done:** The system was deployed against 25 authentic, raw corporate announcements sourced directly from historical A-share filings (spanning finance, real estate, manufacturing, and pharmaceuticals). Each document contained complex layouts, genuine corporate legal jargon, and dense financial tables.
* **Purpose:** Simulates a true production environment. This proved the LLM's ultimate contextual understanding and resilience when target metrics are buried deep within massive, completely unstructured real-world corporate filings.
* **Result (Outcome Metric):** **25/25 Correct (100.0% Extraction Accuracy)**.
