# 🏥 Diabetes Treatment Plan Generator

A Retrieval-Augmented Generation (RAG) system that generates evidence-based diabetes treatment plans by querying a curated knowledge base of authoritative clinical guidelines. Built with LangChain, ChromaDB, GPT-4, and a Gradio web interface.

---

## Overview

This system ingests peer-reviewed clinical guidelines in PDF format, chunks and embeds them into a vector store, and uses GPT-4 to generate patient-specific treatment plans grounded in that evidence base. Clinicians or researchers enter structured patient data through a web UI and receive a comprehensive, citation-supported treatment plan in return.

The system does not hallucinate recommendations — it retrieves relevant passages from authoritative guideline documents and uses them as the sole basis for its output.

---

## Clinical Guidelines Included

| Document | Source |
|---|---|
| Standards of Care in Diabetes — 2024 | American Diabetes Association (ADA) |
| Inpatient Hyperglycemia Management — 2022 | Endocrine Society |
| Type 2 Diabetes in Primary Care | International Diabetes Federation (IDF) |
| Type 1 Diabetes Guidelines (NG17) | NICE |
| Type 2 Diabetes Guidelines (NG28 Summary) | NICE |

> Additional PDFs can be added to the `diabetes_pdfs/` directory and will be automatically ingested on next initialization.

---

## Architecture

```
PDF Guidelines
     │
     ▼
PyPDFLoader → RecursiveCharacterTextSplitter (1000 char chunks, 200 overlap)
     │
     ▼
OpenAI Embeddings → ChromaDB Vector Store
     │
     ▼
MMR Retriever (k=5, fetch_k=20)
     │
     ▼
GPT-4 + System Prompt → Treatment Plan
     │
     ▼
Gradio Web UI
```

**Key design decisions:**
- **MMR (Maximum Marginal Relevance)** retrieval is used instead of pure similarity search to maximize diversity among retrieved chunks and reduce redundancy
- **Temperature = 0** on GPT-4 ensures deterministic, conservative outputs appropriate for a clinical context
- **Source citations** are injected into each retrieved chunk so the LLM can reference them in its output

---

## Requirements

```
python >= 3.10
langchain
langchain-community
langchain-openai
langchain-text-splitters
chromadb
gradio
pypdf
openai
```

Install dependencies:

```bash
pip install langchain langchain-community langchain-openai chromadb gradio pypdf openai
```

---

## Setup

### 1. Clone the repository

```bash
git clone https://github.com/your-username/diabetes-rag.git
cd diabetes-rag
```

### 2. Set your OpenAI API key

Set your key as an environment variable. **Do not hardcode it in the notebook.**

```bash
export OPENAI_API_KEY="your-api-key-here"
```

Or in Python before initializing:

```python
import os
os.environ["OPENAI_API_KEY"] = "your-api-key-here"
```

### 3. Add your PDF guidelines

Place clinical guideline PDFs in a directory named `diabetes_pdfs/`:

```
diabetes_pdfs/
├── ADA_Standards_2024.pdf
├── NICE_Type1_NG17.pdf
├── IDF_Type2_Primary_Care.pdf
└── ...
```

### 4. Run the notebook

Open `RAG_Diabetes_Treatment_Plan_Generator.ipynb` in Jupyter and run all cells. The system will:

1. Load and parse all PDFs
2. Create text chunks
3. Build and persist the ChromaDB vector store
4. Initialize GPT-4
5. Launch the Gradio web interface at `http://127.0.0.1:7860`

---

## Usage

### Via the Gradio Web Interface

The UI collects the following patient inputs:

| Field | Description |
|---|---|
| Age | Patient age in years |
| Gender | Male / Female |
| Diabetes Type | Type 1, Type 2 (new or established), Gestational, Prediabetes |
| HbA1c (%) | Current glycated hemoglobin value |
| BMI | Body mass index |
| Duration | Time since diagnosis |
| Comorbidities | e.g., Hypertension, CKD, CVD |
| Current Medications | Including doses |
| Additional Info | Lifestyle factors, special considerations |

Three preloaded example cases are available:
- **Newly Diagnosed T2DM** — 55M, HbA1c 8.2%, BMI 32, controlled hypertension
- **T2DM with CVD** — 62F, HbA1c 7.8%, prior MI, CKD stage 3a
- **Elderly with CKD** — 78M, HbA1c 8.5%, CKD stage 4, heart failure, hypoglycemia history

### Via Python

```python
from your_module import DiabetesRAGSystem

rag_system = DiabetesRAGSystem(
    openai_api_key=os.environ["OPENAI_API_KEY"],
    pdf_directory="diabetes_pdfs"
)

rag_system.initialize()

patient_query = """
Patient: 60-year-old female
Diagnosis: Type 2 Diabetes (3 years duration)
HbA1c: 7.4%
BMI: 29
Comorbidities: None
Current medications: Metformin 500mg daily
"""

plan = rag_system.generate_treatment_plan(patient_query)
print(plan)
```

### Testing Retrieval

To inspect what guideline content is being retrieved for a given query:

```python
rag_system.test_retrieval("first-line treatment for type 2 diabetes", k=3)
```

---

## Sample Output

**Case: 55-year-old male, newly diagnosed T2DM, HbA1c 8.2%**

> 1. **Glycemic Control:** The patient's HbA1c of 8.2% exceeds the NICE target of 6.5% for patients managed with lifestyle and a single non-hypoglycemic agent. Metformin is recommended as first-line pharmacotherapy, initiated at a low dose and titrated to minimize GI side effects. [Source: NICE NG28]
>
> 2. **Blood Pressure Management:** Current hypertension is controlled on Lisinopril 10mg. IDF guidelines recommend targets of SBP 130–140 mmHg and DBP 80 mmHg for patients with T2DM and hypertension. Continue current therapy with regular monitoring.
>
> 3. **Lifestyle Modifications:** Advise Mediterranean-style diet, structured physical activity, salt restriction, and strong cessation counseling for tobacco use if applicable.
>
> *(additional sections: monitoring schedule, frailty screening, medication review, follow-up cadence)*

---

## Vector Store

The ChromaDB vector store is persisted to `./chroma_db/` after initialization. On subsequent runs, the system can load the existing store rather than re-embedding all documents, reducing startup time and API costs.

---

## Known Issues

- PDFs with malformed headers (e.g., HTML content saved as `.pdf`) will fail to load with a warning and be skipped
- The system requires a valid OpenAI API key with access to `gpt-4`
- Gradio CSS parameter handling changed in v6.0 — pass `css` to `.launch()` rather than `gr.Blocks()` if you encounter a deprecation warning

---

## ⚠️ Disclaimer

This tool is intended for **educational and research purposes only**. It is not a certified clinical decision support system and should **not be used to make real patient care decisions**. Always defer to licensed clinicians and current institutional guidelines for actual treatment planning.

---

## License

MIT License. See `LICENSE` for details.
