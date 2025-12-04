

# **Helix Navigator – Quarter 1 Replication & Extension Project**

**Biomedical AI via LangGraph Workflows, Knowledge Graphs, and Conversational Memory**

This repository replicates and extends the **Helix Navigator** framework originally developed by **Abed El Husseini** and collaborators for the UCSD Halıcıoğlu Data Science Institute.

The goal of this project is to *accurately reproduce* the original system and *extend it* with a new **session-level memory mechanism** that enables multi-turn conversational reasoning over a biomedical Neo4j knowledge graph.

This work was completed for **UCSD DSC 180A – Quarter 1 Replication Project**.

---

## **Project Purpose**

Biomedical research requires integrating knowledge about genes, proteins, diseases, and drugs. Graph databases like Neo4j model this information naturally, but querying them requires skill in Cypher.

Helix Navigator solves this by using a LangGraph workflow to convert natural language questions—e.g.,
**“Which drugs treat hypertension?”**—into executable Cypher queries.

### This replication preserves:

* Neo4j schema, CSV structure, and all data-loading logic
* Baseline LangGraph workflow (classification → extraction → query generation → execution → synthesis)
* Core Streamlit UI design

### This extension adds:

* **Conversation memory** for multi-turn reasoning
* **Pronoun and referent tracking** (e.g., “those drugs”, “the first one”)
* **Per-session history storage**
* **UI enhancements** showing the last ~10 turns

---

## **Implemented Enhancement — Session-Level Memory**

The original Helix Navigator is fully stateless.
This extended version adds **session memory** so the model can reference prior answers during a conversation.

### **Core Enhancements**

#### **1. Extended Workflow State (`WorkflowState`)**

```python
session_id: Optional[str]
history: Optional[List[Dict[str, Any]]]
```

#### **2. In-Memory Turn Recording**

Each turn logs:

* question
* question type
* extracted entities
* generated Cypher query
* result count
* first result rows
* final answer
* errors

History is capped at the last **10** turns.

#### **3. Prompt Augmentation**

Context is injected into:

* question classification
* entity extraction
* query generation

so the agent can resolve expressions like:

* “those drugs”
* “the first answer”
* “filter the previous ones”

#### **4. Streamlit UI Additions**

* Persistent `session_id` in `st.session_state`
* A new collapsible **History** panel
* Workflow calls now pass `session_id` to the agent

#### **5. Updated Test Suite**

Tests now verify:

* history rollover
* session persistence
* correct behavior of memory-driven prompts

---

## **Technology Stack**

* **LangGraph** — Workflow orchestration
* **Neo4j** — Biomedical knowledge graph
* **Anthropic Claude** — Language model
* **Streamlit** — Web interface
* **LangGraph Studio** — Workflow visualization
* **PDM** — Project & dependency manager

---

# **Installation & Quick Start**

### **Requirements**

* Python **3.10+**
* Neo4j Desktop
* PDM (`pip install pdm`)

---

### **1. Install Dependencies**

```bash
pdm install
```

---

### **2. Create Environment Variables**

```bash
cp .env.example .env
```

Edit `.env`:

```env
NEO4J_URI=bolt://localhost:7687
NEO4J_USER=neo4j
NEO4J_PASSWORD=your_password_here
ANTHROPIC_API_KEY=your_key_here
```

---

### **3. Load the Knowledge Graph**

```bash
pdm run load-data
```

---

### **4. Launch the Application**

```bash
pdm run app
```

This launches:

1. Knowledge Graph explanation
2. Cypher query explorer
3. Workflow agent with session memory

---

# **Repository Structure**

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
├── scripts
│   ├── load_data.py
│   └── quickstart.py
├── src
│   ├── agents
│   │   ├── graph_interface.py
│   │   └── workflow_agent.py   ← memory enhancements here
│   └── web
│       └── app.py              ← session_id + history panel
├── tests
│   ├── test_app.py
│   ├── test_graph_interface.py
│   └── test_workflow_agent.py  ← tests for memory
├── pyproject.toml
├── pdm.lock
├── pytest.ini
└── README.md
```

---

# **Code-Level Changes in `workflow_agent.py`**

This section documents **exactly** how your fork differs from the original file in the Helix Navigator repository.

These changes **ONLY** add memory and UI support — no graph logic or schema logic was modified.

### **1. Added Session Fields to WorkflowState**

```diff
+    # Jack's additional fields for session management
+    session_id: Optional[str]
+    history: Optional[List[Dict[str, Any]]]
```

---

### **2. Added Internal Session Store**

```diff
+        # Jack's addition for session management
+        self._session_history: Dict[str, List[Dict[str, Any]]] = {}
```

---

### **3. Added Memory Context Builder**

```diff
+    def _build_memory_context(self, history):
+        """Create a short natural-language summary of recent turns."""
+        ...
```

---

### **4. Classification Now Uses Context**

```diff
- prompt = self._build_classification_prompt(...)
+ base_prompt = self._build_classification_prompt(...)
+ memory_context = self._build_memory_context(history)
+ prompt = <augmented version if memory exists>
```

---

### **5. Query Generation Now Uses Context**

```diff
+ Conversation context:
+ {memory_context or "No prior relevant context."}
```

---

### **6. All Answer Branches Now Record History**

Each path now ends with:

```diff
+ record = {...}
+ sid = state.get("session_id")
+ hist = self._session_history.get(sid, [])
+ hist.append(record)
+ self._session_history[sid] = hist[-10:]
```

---

### **7. `answer_question()` API Updated**

```diff
- def answer_question(self, question):
+ def answer_question(self, question, session_id=None):
+     sid = session_id or "default"
+     prior = self._session_history.get(sid, [])
```

---

# **Running the Application**

### Load Data

```bash
pdm run load-data
```

### Start App

```bash
pdm run app
```

### Debug Workflow Graph

```bash
pdm run langgraph
```

---

# **Example Questions**

### **Single-Turn Queries**

* “Which drugs have high efficacy for treating diseases?”
* “List proteins associated with Alzheimer’s disease.”
* “What genes encode proteins linked to cardiovascular disorders?”

---

### **Multi-Turn Queries (Memory in Action)**

#### **Example 1**

```
Which drugs treat atrial fibrillation?
```

→ returns Diltiazem, Artemisinin, EXP-241

```
Which of those are approved?
```

→ memory resolves **“those”**

#### **Example 2**

```
Which proteins are associated with Alzheimer's?
```

```
Which genes encode the first three?
```

→ memory resolves **“the first three”**

These interactions are **impossible** in the original system.

---

# **Reproducibility Guide**

```bash
pdm run test
pdm run lint
pdm run format
```

---

# **Planned Additions**

| Feature            | Description                                       |
| ------------------ | ------------------------------------------------- |
| Evaluation metrics | Score correctness of generated Cypher + reasoning |
| Workflow logging   | JSON traces for reasoning transparency            |
| Embedding memory   | Long-term semantic memory retrieval               |

---

# **Attribution**

This project is a **replication and extension** of the original **Helix Navigator** created by:

**Abed El Husseini**
Halıcıoğlu Data Science Institute, UC San Diego

Original repository:
[https://github.com/aelhusseini/hdsi_replication_proj_2025](https://github.com/aelhusseini/hdsi_replication_proj_2025)

All schema, data, and baseline design originate from the original work.
All memory architecture, workflow enhancements, and evaluation were developed independently for DSC 180A.

---

# **License**

This project follows the licensing terms of the original Helix Navigator repository.

---

