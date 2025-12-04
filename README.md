# **Helix Navigator – Quarter 1 Replication & Extension Project**

**Biomedical AI via LangGraph Workflows, Knowledge Graphs, and Conversational Memory**

This repository replicates and extends the **Helix Navigator** framework originally developed by **Abed El Husseini** and collaborators for the UCSD Halıcıoğlu Data Science Institute.

The goal of this project is to *accurately reproduce* the original system and *extend it* with a new **session-level memory mechanism** that enables multi-turn conversational reasoning over a biomedical Neo4j knowledge graph.

This work was completed for **UCSD DSC 180A – Quarter 1 Replication Project**.

---

## **Project Purpose**

Biomedical research requires integrating knowledge about genes, proteins, diseases, and drugs. Graph databases like Neo4j model this information naturally, but querying them requires skill in Cypher.

Helix Navigator solves this by using a LangGraph workflow to convert natural language questions—e.g., **“Which drugs treat hypertension?”**—into executable Cypher queries.

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

The original Helix Navigator is fully stateless. This extended version adds **session memory** so the model can reference prior answers during a conversation.

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

## **Installation & Quick Start**

### **Requirements**
* Python **3.10+**
* Neo4j Desktop
* PDM (`pip install pdm`)

### **1. Install Dependencies**
```bash
pdm install
```

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

### **3. Load the Knowledge Graph**
```bash
pdm run load-data
```

### **4. Launch the Application**
```bash
pdm run app
```
This launches:
1. Knowledge Graph explanation
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

## **Code-Level Changes in `src/agents/workflow_agent.py`**

Compared to the original Helix Navigator implementation by Abed El Husseini, this fork introduces **session-aware, conversation-focused behavior** in the `WorkflowAgent`. All graph schema, data loading, and Neo4j logic remain unchanged. The modifications are scoped to how the agent tracks and uses conversational state.

### **1. Extended Workflow State for Sessions**
The `WorkflowState` TypedDict was extended to carry per-session context:
```python
class WorkflowState(TypedDict):
    user_question: str
    question_type: Optional[str]
    entities: Optional[List[str]]
    cypher_query: Optional[str]
    results: Optional[List[Dict]]
    final_answer: Optional[str]
    error: Optional[str]

    # Jack's additional fields for session management
    session_id: Optional[str]
    history: Optional[List[Dict[str, Any]]]
```
This allows the LangGraph workflow to know which session a turn belongs to and what recent turns are available as history.

### **2. In-Memory Session Store**
The `WorkflowAgent` now maintains a simple in-memory session store keyed by `session_id`:
```python
class WorkflowAgent:
    def __init__(self, graph_interface: GraphInterface, anthropic_api_key: str):
        self.graph_db = graph_interface
        self.anthropic = Anthropic(api_key=anthropic_api_key)
        self.schema = self.graph_db.get_schema_info()
        self.property_values = self._get_key_property_values()
        self.workflow = self._create_workflow()

        # Jack's addition for session management
        self._session_history: Dict[str, List[Dict[str, Any]]] = {}
```
Each entry in `_session_history` is a list of turn records for that session.

### **3. Memory Context Builder**
A new helper method summarizes recent turns so the LLM can resolve references like “those drugs” or “the first answer”:
```python
def _build_memory_context(
    self, history: Optional[List[Dict[str, Any]]]
) -> str:
    """Create a short natural-language summary of recent turns for use in prompts."""
    if not history:
        return ""

    # Use the most recent turn
    last = history[-1]

    q = last.get("question") or ""
    qtype = last.get("type") or "unknown_type"
    entities = last.get("entities") or []
    results_count = last.get("results_count", 0)

    context = (
        "Previous turn:\n"
        f"- Question type: {qtype}\n"
        f"- Question: {q}\n"
        f"- Entities: {entities}\n"
        f"- Results count: {results_count}\n"
    )
    return context
```
This does not change the underlying graph query logic; it only creates natural-language context for the prompts.

### **4. Classification Now Uses Conversation Context (Optional)**
The `classify_question` step was updated to conditionally include memory context in its prompt:
```python
def classify_question(self, state: WorkflowState) -> WorkflowState:
    try:
        # Build base classification prompt for the current question
        base_prompt = self._build_classification_prompt(state["user_question"])

        # NEW: incorporate recent conversation history (if any)
        history = state.get("history") or []
        memory_context = self._build_memory_context(history)

        if memory_context:
            prompt = (
                "You are classifying a biomedical question within an ongoing conversation.\n"
                "Use the context below only to help interpret pronouns or references "
                "like 'those diseases' or 'the first answer', but always classify the "
                "current question itself.\n\n"
                f"{memory_context}\n\n"
                f"{base_prompt}"
            )
        else:
            prompt = base_prompt

        state["question_type"] = self._get_llm_response(prompt, max_tokens=20)
    except Exception as e:
        state["error"] = f"Classification failed: {str(e)}"
        state["question_type"] = "general_knowledge"
    return state
```
If no history is present, behavior falls back to the original single-turn prompt.

### **5. Query Generation Uses Memory Context for Follow-Ups**
The `generate_query` stage was also modified so that the LLM can use prior results to interpret referential follow-up questions:
```python
# NEW: incorporate recent conversation history (if any)
history = state.get("history") or []
memory_context = self._build_memory_context(history)

prompt = f"""Create a Cypher query for this biomedical question.

If the current question refers to previous answers (for example, using phrases
like "the first answer", "those diseases", or "those drugs"), use the
conversation context below to resolve what those references point to.

Conversation context:
{memory_context if memory_context else "No prior relevant context."}

Current question: {state['user_question']}
Question type: {question_type}

Schema:
Nodes: {', '.join(self.schema['node_labels'])}
Relations: {', '.join(self.schema['relationship_types'])}

{property_info}
{relationship_guide}

Entities detected in the current question: {state.get('entities', [])}

Write a SINGLE Cypher query that answers ONLY the current question.
Use MATCH, WHERE with CONTAINS for filtering, RETURN, and LIMIT 10.
IMPORTANT:
- Use only property names and relationship types from the schema above.
- Use IN [value1, value2] when filtering over discrete property values.
Return only the Cypher query, with no explanation or commentary."""
```
Again, if there is no prior history, the prompt simply contains “No prior relevant context.” and behaves like the baseline.

### **6. Recording Turns into Session History**
At the end of `format_answer`, each turn is now recorded into `_session_history` and attached back to the state:
```python
record = {
    "question": state.get("user_question"),
    "type": state.get("question_type"),
    "entities": (state.get("entities") or [])[:8],
    "cypher": (
        state.get("cypher_query")[:500]
        if state.get("cypher_query")
        else None
    ),
    "results_count": len(state.get("results") or []),
    "error": state.get("error"),
}

sid = state.get("session_id") or "default"
hist = self._session_history.get(sid, [])
hist.append(record)
if len(hist) > 10:
    hist = hist[-10:]  # sliding window of recent turns
self._session_history[sid] = hist
state["history"] = hist
return state
```
This recording logic is applied consistently in all branches:
* when there is an error,
* when the question is general_knowledge,
* when there are no results,
* and when results are successfully returned.

### **7. Session-Aware answer_question API**
Finally, `answer_question` now supports multi-turn sessions explicitly:
```python
def answer_question(
    self,
    question: str,
    session_id: Optional[str] = None
) -> Dict[str, Any]:
    sid = session_id or "default"
    prior = self._session_history.get(sid, [])

    initial_state = WorkflowState(
        user_question=question,
        question_type=None,
        entities=None,
        cypher_query=None,
        results=None,
        final_answer=None,
        error=None,
        session_id=sid,
        history=prior[-10:],  # seed with last turns
    )

    final_state = self.workflow.invoke(initial_state)

    return {
        "answer": final_state.get("final_answer", "No answer generated"),
        "question_type": final_state.get("question_type"),
        "entities": final_state.get("entities", []),
        "cypher_query": final_state.get("cypher_query"),
        "results_count": len(final_state.get("results", [])),
        "raw_results": final_state.get("results", [])[:3],
        "error": final_state.get("error"),
        "history": final_state.get("history", []),  # expose for UI/tests
    }
```
If `session_id` is not provided, the agent defaults to a `"default"` session, preserving prior behavior while enabling explicit session scoping when the UI (or tests) pass a specific ID.

---

## **Running the Application**
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

## **Example Questions**
### **Single-Turn Queries**
* “Which drugs have high efficacy for treating diseases?”
* “List proteins associated with Alzheimer’s disease.”
* “What genes encode proteins linked to cardiovascular disorders?”

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

## **Reproducibility Guide**
```bash
pdm run test
pdm run lint
pdm run format
```

---

## **Planned Additions**
| Feature            | Description                                       |
|--------------------|---------------------------------------------------|
| Evaluation metrics | Score correctness of generated Cypher + reasoning |
| Workflow logging   | JSON traces for reasoning transparency            |
| Embedding memory   | Long-term semantic memory retrieval               |

---

## **Attribution**
This project is a **replication and extension** of the original **Helix Navigator** created by:

**Abed El Husseini**
Halıcıoğlu Data Science Institute, UC San Diego

Original repository:
[https://github.com/aelhusseini/hdsi_replication_proj_2025](https://github.com/aelhusseini/hdsi_replication_proj_2025)

All schema, data, and baseline design originate from the original work.
All memory architecture, workflow enhancements, and evaluation were developed independently for DSC 180A.

---

## **License**
This project follows the licensing terms of the original Helix Navigator repository.
```