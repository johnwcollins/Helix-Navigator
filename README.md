# **Helix Navigator – Quarter 1 Replication & Extension Project**

**Biomedical AI via LangGraph Workflows, Knowledge Graphs, and Conversational Memory**

This repository replicates and extends the **Helix Navigator** framework originally developed by **Abed El Husseini** and collaborators for the UCSD Halıcıoğlu Data Science Institute.

The purpose of this project is to **extend** the system with a new **session-level memory mechanism** that enables multi-turn conversational reasoning over a biomedical knowledge graph.

This work was completed as part of the **UCSD DSC 180A – Quarter 1 Replication Project**.

---

## **Project Purpose**

Biomedical research requires integrating information about genes, proteins, diseases, and drugs. Knowledge graphs such as Neo4j represent these relationships effectively, but querying them requires specialized knowledge of languages such as Cypher.

Helix Navigator enables natural language interaction with structured biomedical knowledge. Using a LangGraph workflow, the system converts questions such as *“Which drugs treat hypertension?”* into executable Cypher queries.

This replication preserves:

- Neo4j schema and data-loading logic  
- Baseline LangGraph workflow (classification → extraction → generation → execution → synthesis)

This extension introduces:

- **Conversation continuity**
- **Context-aware reference resolution**
- **Session persistence through Streamlit**

---

## **Implemented Enhancement — Session-Level Memory**

The original Helix Navigator is fully stateless: every query is treated independently. This extended version adds **session memory**, enabling multi-turn conversational reasoning.

### **Core Enhancements**

#### 1. Extended Workflow State

```python
session_id: str
history: List[Dict[str, Any]]
```

#### 2. Turn Recording

Each interaction stores:

- question  
- classified type  
- extracted entities  
- Cypher query  
- result count  
- first result rows  
- final answer  
- errors  

History is capped at ~10 turns.

#### 3. Prompt Augmentation

Relevant history is injected into:

- `classify_question`
- `generate_query`

so the model can resolve references like:

- “those drugs”
- “the first one”
- “filter the previous results”

#### 4. Streamlit UI Updates

- Persistent session ID in `st.session_state`
- A collapsible “History” panel showing recent turns
- Passes session ID into `agent.answer_question`

#### 5. Updated Tests

- history rollover  
- session persistence  
- memory-driven behavior  

---

## **Technology Stack**

- **LangGraph** — Multi-step workflow orchestration  
- **Neo4j** — Biomedical graph database  
- **Anthropic Claude** — LLM powering reasoning  
- **Streamlit** — Interactive web UI  
- **LangGraph Studio** — Visual workflow debugger  
- **PDM** — Environment and dependency manager  

---

## **Installation & Quick Start**

### **Requirements**

- Python **3.10+**
- **Neo4j Desktop**
- **PDM**

```bash
pip install pdm
```

---

### **1. Install Dependencies**

```bash
pdm install
```

---

### **2. Set Up Environment Variables**

```bash
cp .env.example .env
```

Edit `.env`:

```env
NEO4J_URI=bolt://localhost:7687
NEO4J_USER=neo4j
NEO4J_PASSWORD=your_password_here
ANTHROPIC_API_KEY=your_api_key_here
```

---

### **3. Load the Biomedical Graph**

```bash
pdm run load-data
```

Loads all CSVs under `/data`.

---

### **4. Start the Web Application**

```bash
pdm run app
```

This launches:

1. Knowledge Graph overview  
2. Cypher query explorer  
3. Workflow agent with session memory  

---

## **Repository Structure**

```
.
├── data
│   ├── diseases.csv
│   ├── drug_disease_treatments.csv
│   ├── drug_protein_targets.csv
│   ├── drugs.csv
│   ├── genes.csv
│   ├── protein_disease_associations.csv
│   └── proteins.csv
├── docs
│   ├── foundations-and-background.md
│   ├── getting-started.md
│   ├── reference.md
│   └── technical-guide.md
├── langgraph-studio
│   ├── langgraph_studio.py
│   └── langgraph.json
├── pdm.lock
├── pyproject.toml
├── pytest.ini
├── README.md
├── scripts
│   ├── load_data.py
│   └── quickstart.py
├── src
│   ├── agents
│   │   ├── graph_interface.py
│   │   └── workflow_agent.py
│   └── web
│       └── app.py
└── tests
    ├── test_app.py
    ├── test_graph_interface.py
    └── test_workflow_agent.py
```

---

## **Running the Application**

### Load data

```bash
pdm run load-data
```

### Run the app

```bash
pdm run app
```

### Visual debugging with LangGraph Studio

```bash
pdm run langgraph
```

### Development Commands

```bash
pdm run lint
pdm run format
pdm run test
```

---

## **Example Questions**

### **Single-turn**

- “Which drugs have high efficacy for treating diseases?”
- “List proteins associated with Alzheimer’s disease.”
- “What genes encode proteins linked to cardiovascular disorders?”

### **Multi-turn (session memory functionality)**

1.  
   ```
   Which drugs treat atrial fibrillation?
   ```
   → returns 3 drugs  
   ```
   Which of those are approved?
   ```
   → memory resolves “those”

2.  
   ```
   Which proteins are associated with Alzheimer's?
   ```
   → returns list  
   ```
   Which genes encode the first three?
   ```
   → memory resolves “the first three”

These interactions were impossible in the original system.

---

## **Reproducibility Guide**

```bash
pdm run test
pdm run lint
pdm run format
```

All results are deterministic from `pdm.lock`.

---

## **Planned Additions**

| Feature            | Description                                       |
| ------------------ | ------------------------------------------------- |
| Evaluation metrics | Score correctness of generated Cypher + reasoning |
| Workflow logging   | Structured JSON for reasoning traceability        |
| Embedding memory   | Semantic retrieval for longer conversations       |

---

## **Attribution**

This project is a **replication and extension** of the original **Helix Navigator** framework created by:

**Abed El Husseini**, Halıcıoğlu Data Science Institute, UC San Diego  
Original repository:  
https://github.com/aelhusseini/hdsi_replication_proj_2025

All schema, data, and baseline architecture originate from that work.  
All session-memory extensions and evaluation analyses were developed independently for the UCSD DSC 180A project.

---

## **License**

This project follows the licensing terms of the original Helix Navigator repository.
