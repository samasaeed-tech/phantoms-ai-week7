# W7D1: Vector Database Fundamentals & ChromaDB

## 1. Vector DB vs. Traditional SQL Database
* **SQL Database:** Relies on exact keyword matching, structured schemas, and filters (e.g., `WHERE topic = 'programming'`). If a query uses synonyms or conceptual phrasing that doesn't match database columns or terms, it fails to retrieve results.
* **Vector Database (Chroma):** Stores text as high-dimensional numerical embeddings and performs **Approximate Nearest Neighbor (ANN)** searches based on semantic distance. It retrieves documents sharing conceptual closeness even if no words overlap.

---

## 2. Query Evaluation Table

| Query | Is First Result Correct? | What Was Returned (if incorrect)? |
| :--- | :--- | :--- |
| **Q1 (Python values storage)** | Yes | Python lists mutable sequences document. |
| **Q2 (Workout injury prevention)** | Yes | Stretching before exercise document. |
| **Q3 (Bread density/hardness)** | Yes | Letting dough rest allows gluten relaxation document. |
| **Q4 (Merging database tables)** | Yes | SQL JOINs combine rows from two tables document. |
| **Q5 (Rocket to Mars - Out of Domain)** | N/A (Semantic search always returns closest items) | Returned unrelated documents (e.g., Python/Cooking) with very high distances. |

---

## 3. The Q5 Phenomenon & RAG Thresholds
* **What happened in Q5?** Because Vector DBs always calculate mathematical distances to find the "nearest" neighbor in vector space, asking about "deploying a rocket to Mars" still returned documents (like programming or cooking tips) because it had to pick *something*.
* **Why is this critical for RAG?** If a RAG system blindly sends these distant retrieved chunks to an LLM, the LLM will hallucinate or force an irrelevant answer. 
* **Proposed Threshold Solution:** By observing that valid semantic matches usually have low distances (e.g., `< 1.0` depending on the metric), we can set a **Distance Threshold** (e.g., `max_distance = 1.2`). If the closest retrieved document exceeds this threshold, the system halts and outputs: *"I don't have information about this in my knowledge base."*

---

## 4. Chroma Configuration & Metadata Filtering
* **Default Distance Metric:** Chroma uses **L2 (Squared Euclidean Distance)** by default for its collections.
* **Metadata Filter Impact:** Running Q2 with `where={"topic": "fitness"}` successfully restricted the search space exclusively to fitness documents, ensuring no cross-domain contamination (e.g., preventing programming or cooking tips from showing up).
* **Persistence & Duplication Prevention:** Using `PersistentClient(path="./chroma_db")` ensures data survives program restarts, and checking `existing_ids` prevents duplicate document insertions upon multiple script executions.
