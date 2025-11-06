`````markdown
# Helix Navigator – Quarter 1 Replication Project

**Biomedical AI through LangGraph and Knowledge Graph Workflows**

This repository replicates and extends the **Helix Navigator** framework as part of my UCSD DSC 180A Quarter 1 Project.  
The original system was developed by **Abed El Husseini** and collaborators in the [hdsi_replication_proj_2025](https://github.com/aelhusseini/hdsi_replication_proj_2025) repository.  
This version introduces **session-level memory** and related workflow enhancements for conversation persistence and evaluation.

---

## Project Purpose

The project explores how language-model reasoning can integrate with structured biomedical knowledge represented in graph form.  
It reproduces the Helix Navigator baseline and extends it with conversation continuity and reproducibility features while leaving all Neo4j schema and data logic untouched.

---

## Implemented Enhancement – Session-Level Memory

This version introduced **session-level memory** to the Helix Navigator workflow.

### Core Changes

1. **State model expanded**  
   Added `session_id` and `history` fields to `WorkflowState` so each run can persist context and prior turns.

2. **History recording**  
   In `format_answer`, appended a compact record of each completed interaction — question, type, entities, Cypher query, result count, error, timestamp — capped at ≈ 10 entries.

3. **WorkflowAgent interface**  
   Updated `answer_question` to accept a `session_id`, reuse existing `history`, and return the updated record list with every response.

4. **Streamlit integration**  
   Stored a persistent `session_id` in `st.session_state`; passed it into `agent.answer_question`; added a collapsible **History** panel in the UI showing recent queries and answers.

5. **Tests**  
   Added coverage verifying:  
   • history list exists and rolls over correctly  
   • new entries reflect the latest question  
   • session reuse preserves context across runs  

No database schema or Neo4j logic changed; all persistence remains in memory through Streamlit session state.

---

## Quick Start

**Requirements:** Python 3.10+, Neo4j Desktop, PDM

```bash
git clone https://github.com/johnwcollins/helix-navigator-replication.git
cd helix-navigator-replication
pdm install
cp .env.example .env
pdm run load-data
pdm run app
````

To validate reproducibility:

```bash
pdm run test
pdm run lint
pdm run format
```

---

## Repository Structure

```
├── data/                     # Biomedical CSV datasets
├── docs/                     # Foundations, setup, reference, technical guides
├── langgraph-studio/         # Visual debugging assets
├── scripts/                  # Data loaders and quickstart scripts
├── src/
│   ├── agents/
│   │   ├── graph_interface.py
│   │   └── workflow_agent.py     # updated with session memory logic
│   └── web/
│       └── app.py                # updated with history panel + session_id
└── tests/                    # Expanded unit tests for history + context
```

---

## Technology Stack

* **LangGraph** – Workflow orchestration
* **Neo4j** – Biomedical knowledge graph
* **Anthropic Claude** – Language reasoning
* **Streamlit** – Interactive web UI
* **LangGraph Studio** – Workflow visualization

---

## Reproducibility Guide

1. **Data Access**
   Launch Neo4j Desktop and load the CSVs under `data/`.

2. **Dependencies**
   Managed through `pdm.lock` for deterministic builds.

3. **Execution**

   ```bash
   pdm run load-data
   pdm run app
   ```

---

## Planned Additions

| Feature                | Description                                                                     |
| ---------------------- | ------------------------------------------------------------------------------- |
| **Evaluation Metrics** | Add node- and system-level scoring for workflow accuracy and retrieval quality. |
| **Workflow Logging**   | Structured JSON traces for agentic reasoning steps.                             |

---

## Example Queries

* “Which approved drugs target proteins associated with Alzheimer’s disease?”
* “What genes encode proteins linked to cardiovascular disorders?”
* “Which drugs show strong efficacy across related disease classes?”

---

## Attribution

Original Helix Navigator materials and architecture were created by **Abed El Husseini** in the
[hdsi_replication_proj_2025](https://github.com/aelhusseini/hdsi_replication_proj_2025) repository.
This repository represents my independent **replication and extension** for the UCSD DSC 180A Quarter 1 Project Checkpoint.

```
```
