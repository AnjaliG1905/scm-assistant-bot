# SCM Assistant Bot 🤖

A Retrieval-Augmented Generation (RAG) chatbot built on **Flowise** that answers natural-language questions about a supplier network — covering purchase order performance, compliance scores, risk classifications, and governance policy obligations.

---

## 🔗 Public Chatbot URL

**[https://cloud.flowiseai.com/chatbot/2a0eacab-b74c-4db5-9995-b2cd5beaa373](https://cloud.flowiseai.com/chatbot/2a0eacab-b74c-4db5-9995-b2cd5beaa373)**

> Title: **SCM Assistant**
> Welcome message: *"Hello! I'm your Supply Chain Management Assistant. Ask me anything about supplier performance, compliance, risk levels, or governance policy."*

---

##  Models Used

| Role | Model |
|------|-------|
| LLM | `gpt-4o` (OpenAI) |
| Embeddings | `text-embedding-3-large` (OpenAI) |
| Vector Store | Pinecone (`scm-assistant-v3`, Top K = 120) |

---

## 📂 Data Sources

| File | Description | Chunks |
|------|-------------|--------|
| `SupplyChain_Governance_Policy_v3.2.pdf` | 10-section supplier governance policy — tier thresholds, SLAs, penalties, audit rules, disruption response procedures | 19 |
| `supplier_summaries_v2.txt` | Structured supplier-level summaries derived from `supplier_performance_data.csv` — 2,000 POs across 116 suppliers, 27 columns | 118 |

**Total chunks upserted: 137**

---

## ⚙️ Chunk Configurations Tried

### Config 1 — Fine-grained (Final choice)
| Parameter | Value |
|-----------|-------|
| Chunk Size | 600 |
| Chunk Overlap | 0 |
| Splitter | RecursiveCharacterTextSplitter |

**Result:** Higher chunk count, more precise retrieval for policy clauses and individual supplier metrics. Reduced hallucination on numeric threshold questions. Selected as the production config.

---

### Config 2 — Coarse
| Parameter | Value |
|-----------|-------|
| Chunk Size | 1500 |
| Chunk Overlap | 150 |
| Splitter | RecursiveCharacterTextSplitter |

**Result:** Fewer, broader chunks. Worked well for multi-step reasoning questions but occasionally retrieved irrelevant supplier blocks when querying specific policy sections. Discarded in favour of Config 1.

---

## ❓ Sample Q&A — Verbatim Chatbot Answers

> ⚠️ **Note:** These answers are computed directly from the provided `supplier_performance_data.csv` and `SupplyChain_Governance_Policy_v3.2.pdf` loaded into this chatbot. They differ from the sample expected answers in the task brief, which appear to have been generated from a different version of the CSV data.

---

### Q1 — Tier-3 Suppliers with Active Disruption Flags

**Question:** Which Tier-3 suppliers have an active disruption flag, and what response level applies per policy?

**Answer:**
The Tier-3 suppliers with active disruption flags are:

1. Longhua Polymer Works
2. Yucatan Polymer Mfg
3. Buenos Aires Pack
4. Visayas Textile Co
5. Maghreb Castworks
6. Deltaforge Vietnam
7. Sibertek Molding
8. MeKong Pack Co
9. Lima Polymer SA
10. Helios Pack Greece
11. Bogota Pack Ltda
12. Orinoco Pack SAPI

The response levels per Policy §9 are:
- **Level 2 — Manage** for Medium Risk suppliers
- **Level 3 — Activate** for High Risk suppliers

---

### Q2 — Volume Rebate Program Qualification

**Question:** Which suppliers qualify for the annual Volume Rebate Program and how many are there?

**Answer:**
To qualify for the annual Volume Rebate Program, a supplier must meet all of the following criteria per Policy §4.2:
- Be a **Tier-1** supplier
- Have an **OTD Rate ≥ 93.0%**
- Have a **Defect Rate < 0.5%**
- Have a **Sustainability Score ≥ 85**
- All criteria must be met simultaneously for a full calendar year

Upon reviewing the supplier records:

- **Formosa Circuit Co (SUP-034)** — OTD: 96.0%, Defect: 0.29%, Sustainability: 85 ✅ **Qualifies**
- Lumivex Semiconductor NL (SUP-054) — Sustainability Score: 70 ❌ Does not meet sustainability requirement
- Valdoro Special Alloys (SUP-066) — OTD: 91.1% ❌ Does not meet OTD requirement
- Kinki Precision Corp (SUP-029) — Defect Rate: 0.5% ❌ Does not meet defect rate requirement
- Tasman Circuit Boards (SUP-043) — Defect Rate: 1.13% ❌ Does not meet defect rate requirement

**Final Answer: Only 1 supplier qualifies — Formosa Circuit Co.**

---

### Q3 — Region with Highest Total PO Value

**Question:** Which region has the highest total PO value, and does it breach the concentration limit?

**Answer:**
The **APAC** region has the highest total PO value at **$131,620,356.14**. It does **not** breach the 45% regional concentration cap per Policy §5.3.

---

### Q4 — Supplier Watch List (SWL) Status

**Question:** Which suppliers are on Supplier Watch List (SWL) status and what does it restrict?

**Answer:**
The suppliers on Supplier Watch List (SWL) status are:

1. Longhua Polymer Works
2. Lima Polymer SA
3. Visayas Textile Co
4. DaNang Metal Works

SWL status restricts **new PO issuance to 20% of prior quarter volume** per Policy §3.4.

---

### Q5 — Product Category with Highest Average Defect Rate

**Question:** Which product category has the highest average defect rate and does it exceed the Tier-2 limit?

**Answer:**
The product category with the highest average defect rate is **Packaging Materials** at **1.9092%**. This does **not** exceed the Tier-2 ceiling of **2.50%** per Policy §3.2 — so no breach is triggered.

---

## 🔧 What I Would Improve

1. **Use the raw CSV directly** — Instead of pre-processing the CSV into a summary text file, a future iteration would use a structured tool (SQL agent or Pandas tool node in Flowise) to query the CSV directly. This eliminates summarisation loss and guarantees numeric accuracy.

2. **Hybrid search** — Combine dense vector search with BM25 sparse retrieval (Pinecone hybrid mode) to improve recall on exact supplier names, IDs, and numeric thresholds.

3. **Evaluation harness** — Build an automated eval suite (e.g. with Ragas or LangSmith) using the 5 provided Q&A pairs as ground truth, so chunk config and Top K changes can be benchmarked objectively rather than manually.


---

## 📁 Repository Structure

```
scm-assistant-bot/
├── scm_assistant.json          # Exported Flowise chatflow
├── supplier_summaries_v2.txt   # Structured supplier data (derived from CSV)
├── screenshots/                # Step-by-step screenshots
│   ├── 01_document_store.png
│   ├── 02_chunk_config_1.png
│   ├── 03_chunk_config_2.png
│   ├── 04_chatflow_canvas.png
│   ├── 05_share_chatbot.png
│   └── 06_sample_qa.png
├── .gitignore
└── README.md
```

---

## 🔒 Security

API keys are excluded via `.gitignore`. No secrets are committed to this repository.

---

*Trinamix INC · Junior AI Engineer Hiring Task · Ref: TX-JrAI-003*

