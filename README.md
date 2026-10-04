# PE6201 End-of-Course Project: Eval-Driven Financial Data Extraction Pipeline

**Author:** Guo Shuhan (Sylvia)
**Course:** PE6201

## 1. Project Overview & Evolution

This project focuses on automating the extraction of checkable, empirical financial data from unstructured corporate share placement announcements (《配股发行预案》).

Based on instructor feedback during Milestone 1, the project architecture was deliberately evolved. The initial proposal aimed to use a generalized RAG approach to also predict or summarize subsequent stock price movements. Following the feedback, the scope was refined to strictly focus on what the LLM does best: extracting deterministic numbers from unstructured text. Stock price movements are left to standard programmatic computation using historical price data arrays, rather than relying on LLM generation.

The pipeline now strictly focuses on extracting two verifiable metrics with a single right answer:

1. **Placement Size (配股比例)**
2. **Discount Rate (发行折价率)**

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

### Phase 1: Pipeline Validation (Ideal Environment)

* **What was done:** The extraction function was tested using 25 extremely short, artificially simplified mock announcements (e.g., "【浦发银行配股预案】配股比例0.3，折价率0.55").
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

---

---

# PE6201 期末项目：基于评估驱动的金融数据抽取管线

**作者：** 郭书含 (Sylvia)
**课程：** PE6201

## 1. 项目概述与演进

本项目专注于从非结构化的上市公司《配股发行预案》中，自动化提取可核查的实证金融数据。

基于 Milestone 1 阶段的老师反馈，项目架构进行了针对性的演进。最初的提案计划使用宽泛的 RAG 方法来预测或总结后续的股价波动。根据反馈，项目范围被精确收缩，严格聚焦于大模型最擅长的领域：从非结构化文本中提取确定性的数字。至于股价的涨跌，则完全交由常规程序使用历史价格数据进行精确计算，不再依赖大模型去生成。

目前的管线严格专注于提取两个只有一个正确答案的可核查指标：

1. **配股比例 (Placement Size)**
2. **发行折价率 (Discount Rate)**

## 2. 数据与基准 (Ground Truth)

数据集采用了一个**冻结的 25 个 A 股历史配股案例集合**。
为了建立严谨的评估基准，在管线首次运行*之前*，这 25 个案例真实的配股比例和折价率已被人工手抄录入至一个静态的 CSV 文件中（即 Ground Truth）。

## 3. 结果指标 (Outcome Metrics)

按照指导建议，由于缺乏人工计时的基准，诸如“减少分析时间”等主观指标已被彻底剔除。
本项目的唯一技术与业务结果指标为：**基于这 25 个冻结案例的抽取准确率（以具体正确数量呈现）**。

## 4. 技术栈与架构

为了实现零失误的抽取数量，本项目采用了现代 AI 技术栈：

* **基础大模型：** 通过 API 调用 `gpt-4o-mini`，处理原始公司财报复杂的阅读理解任务。
* **Pydantic 结构化输出：** 用严格的 Python 数据模式（`PlacementExtraction`）取代了脆弱的提示词工程。强制大模型返回确定性的 JSON。
* **实体对齐强制机制：** 提示词中动态注入了特定的 `stock_name`（股票简称），防止模型返回公司全称，确保与自动评分脚本实现 1:1 精准匹配。
* **评估驱动测试台 (Eval-Driven Harness)：** 一个自动化脚本，将大模型的输出（`model_outputs.jsonl`）与人工手抄的基准 CSV 进行交叉比对，并采用 `< 0.01` 的容差以规避浮点数计算伪影。

## 5. 三阶段渐进式评估 (Evals)

为了证明系统的最终可靠性，管线接受了三阶段渐进式压力测试，从最小可行性切片开始，逐步扩展至真实世界的复杂性。

### 阶段一：管线验证（理想环境）

* **工作内容：** 使用 25 条极其简短、人为简化的模拟公告（例如：“【浦发银行配股预案】配股比例0.3，折价率0.55”）测试了抽取函数。
* **测试目的：** 验证代码的基础管道连通性。
* **测试结果：** **25/25 正确**。这证明了在没有文本噪音的情况下，API 连接、JSON 契约和打分脚本的运作完美无缺。

### 阶段二：中间态仿真（算法噪音）

* **工作内容：** 管线处理了 25 篇通过程序生成的 5000 字长篇仿真公告。这些文本注入了模拟的金融噪音、标准法律免责声明以及具有迷惑性的诱饵数字。
* **测试目的：** 测试基础模型过滤基础算法干扰项，并严格遵守 JSON 模式约束的能力。
* **测试结果：** **25/25 正确**。

### 阶段三：真实世界压力测试（大海捞针）

* **工作内容：** 将系统直接应用于 25 份真实的、未经加工的 A 股历史原版公告（涵盖金融、地产、制造和医药等行业）。每份文档都包含复杂的排版、纯正的商业法律术语和密集的财务表格。
* **测试目的：** 模拟真实的生产环境。证明当目标指标被深埋在海量、完全非结构化的真实世界企业财报中时，大模型的终极上下文理解能力和抗干扰韧性。
* **测试结果 (Outcome Metric)：** **25/25 正确 (100.0% 抽取准确率)**。
