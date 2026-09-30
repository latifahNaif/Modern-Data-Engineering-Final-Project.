#  Customer Support Ticke

**Modern Data Engineering for AI Systems – Final Project (SDAIA Academy)**
- ✅***This project was created as part of the "Modern Data Engineering for AI Systems program" at SDAIA Academy.***

---

##  Project Overview
This project demonstrates an end-to-end Modern Data Engineering pipeline designed for managing customer support tickets. The system ingests raw support tickets, passes them through an automated **Quality Gate**, isolates invalid data into a **Quarantine Delta Lake**, and writes clean, trusted data into a **Gold Delta Lake Table**. The trusted data is then indexed into a Vector Database to power a **RAG-based Support Assistant**.

---

##  Problem Description
Customer support ticket datasets often contain duplicates, missing values, invalid priority levels, or incomplete resolution descriptions. Ingesting unverified data directly into LLM assistants leads to inaccurate AI responses and poor customer experience. This project solves this issue by enforcing data quality, schema integrity, and automated batch quarantine.

---

##  Data Source
The pipeline uses synthetic multi-category support ticket data (Billing, Login, Shipping, Returns) containing realistic data quality issues:
- `tickets_good.csv`: Clean dataset (low error rate <= 5%) -> **PASS Case**
- `tickets_bad.csv`: Corrupted dataset (high error rate > 5%) -> **FAIL Case**

---

##  Workflow & Architecture

```text
                       +-----------------------+
                       |  Raw Ticket Ingestion |
                       |      (CSV / Data)     |
                       +-----------------------+
                                   |
                                   v
                       +-----------------------+
                       |   PySpark Batch Load  |
                       +-----------------------+
                                   |
                                   v
                       +-----------------------+
                       |  Data Quality Checks  |
                       | - Completeness        |
                       | - Uniqueness          |
                       | - Validity            |
                       | - Business Rules      |
                       +-----------------------+
                                   |
                           +-------+-------+
                           |               |
                       [ FAIL ]         [ PASS ]
                           |               |
                           v               v
               +-------------------+   +-------------------+
               | Quarantine Zone   |   |  Clean Delta Lake |
               | (Delta Lake)      |   |  (Gold Table)     |
               +-------------------+   +-------------------+
                                               |
                                               v
                                       +-------------------+
                                       |  RAG AI Assistant |
                                       | (ChromaDB + LLM)  |
                                       +-------------------+
```

---

##  Data Quality Checks & Quality Gate

To ensure downstream reliability, the pipeline enforces strict Data Quality (DQ) validation before writing to storage:

### 1. Implemented Data Quality Rules
* **Completeness:** `question` and `resolution` fields must not be `NULL` or empty.
* **Uniqueness:** `ticket_id` must be unique per batch.
* **Validity:** `priority` must belong to the approved set: `{low, medium, high}`.
* **Accuracy / Business Rule:** `resolution` text length must be >= 10 characters.

### 2. Quality Gate Logic
* **PASS (<= 5% Error Rate):** Valid rows move to the **Clean Delta Table**, while bad rows are isolated into **Quarantine**.
* **FAIL (> 5% Error Rate):** The entire batch is rejected and routed to **Quarantine** along with failure reason logs.

---

##  AI / RAG Pipeline Engine

* **Embeddings:** Multi-lingual MiniLM (`paraphrase-multilingual-MiniLM-L12-v2`)
* **Vector Database:** ChromaDB
* **LLM & Retrieval Output:** Fetches contextually similar past resolutions to generate precise, grounded support answers while eliminating hallucinations.

---

##  Pipeline Results

| Batch | Input File | Quality Gate Decision | Action Taken |
| :--- | :--- | :---: | :--- |
| **Batch 1** | `tickets_good.csv` | **PASS** (1% Error Rate) | Integrated into Clean Delta Lake & Vector DB |
| **Batch 2** | `tickets_bad.csv` | **FAIL** (High Error Rate) | Isolated completely into Quarantine Delta Table |

---

##  Technologies Used

* **Language:** Python
* **Data Processing Engine:** PySpark 3.5.3
* **Lakehouse Architecture:** Delta Lake 3.2.1
* **Vector Database:** ChromaDB
* **Embeddings & LLM Integration:** Sentence-Transformers & Anthropic Claude API

---

##  How to Run the Project

1. Open **Google Colab** or a local **Jupyter Notebook** environment.
2. Install the required dependencies:

```bash
pip install pyspark==3.5.3 delta-spark==3.2.1 sentence-transformers chromadb anthropic
```

3. Run all notebook cells sequentially.

---

##  Future Improvements

- [ ] **Real-Time Streaming:** Integrate Structured Streaming via Apache Kafka for live ticket streams.
- [ ] **Data Governance & Privacy:** Implement automated PII masking for personal data privacy (PDPL compliance).
- [ ] **User Interface:** Build an interactive web interface using Streamlit for customer interactions.
