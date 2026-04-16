---

# 🧠 Hybrid GraphRAG System using Neo4j + Vector Search

> End-to-end Graph Retrieval-Augmented Generation (GraphRAG) pipeline combining Neo4j graph traversal with vector similarity search for hybrid semantic + structured query answering.

---

## 🚀 Overview

This project implements a **Hybrid GraphRAG architecture** that integrates:

* 📊 Structured graph data (Neo4j AuraDB)
* 🔎 Vector similarity search (Sentence Transformers)
* 🔗 Graph traversal (Cypher)
* 🧠 Hybrid semantic + symbolic filtering
* 📈 Retail analytics queries

The system models retail customer data and enables advanced queries such as:

* Top spending customers
* Customers by state
* Most purchased products
* High-rated reviews
* Semantic queries like:

```text
"Show durable furniture reviews from California"
```

---

## 🏗 Architecture

```
User Question
      ↓
Rule-Based Router (Intent Detection)
      ↓
 ┌──────────────────────────────┐
 │ Structured Query (Cypher)   │
 │ OR                           │
 │ Hybrid Retrieval             │
 │   → Vector Search            │
 │   → Graph Traversal          │
 │   → Structured Filtering     │
 └──────────────────────────────┘
      ↓
Neo4j AuraDB
      ↓
Ranked Results
```

---

## 📂 Dataset

Retail Customer & Transaction Dataset (Kaggle)

Loaded:

| Entity               | Count   |
| -------------------- | ------- |
| Customers            | 5,000   |
| Transactions         | 32,295  |
| Products             | 165     |
| Reviews              | 1,000   |
| Relationships        | 30,000+ |
| Embedding Dimensions | 384     |

---

## 🗄 Graph Schema

### Nodes

* `Customer`
* `Transaction`
* `Product`
* `Review`
* `Campaign`
* `SupportTicket`
* `Interaction`

### Relationships

* `(Customer)-[:PURCHASED]->(Transaction)`
* `(Transaction)-[:CONTAINS]->(Product)`
* `(Customer)-[:WROTE]->(Review)`
* `(Review)-[:ABOUT]->(Product)`

---

## 🧠 Vector Search Layer

* Model: `all-MiniLM-L6-v2`
* Embedding size: 384 dimensions
* Similarity: Cosine
* Stored inside Neo4j vector index

Example hybrid query:

```cypher
CALL db.index.vector.queryNodes(
    'review_embedding_index',
    10,
    $embedding
)
YIELD node AS review, score

MATCH (review)<-[:WROTE]-(c:Customer)
MATCH (review)-[:ABOUT]->(p:Product)

WHERE c.state = $state
AND review.rating >= 4
AND ($category IS NULL OR p.product_category = $category)

RETURN review.review_title,
       p.product_name,
       c.full_name,
       score
ORDER BY score DESC
LIMIT 5
```

---

## 🔥 Key Features

### ✅ Structured Graph Analytics

* Top spending customers
* Most purchased products
* Customer segmentation by state
* Average ratings per product

### ✅ Semantic Retrieval

* Vector similarity over review text
* Ranked by cosine similarity

### ✅ Hybrid GraphRAG

* Embedding search + Graph traversal
* State-based filtering
* Category detection
* Rating threshold filtering
* Ranked semantic results

---

## 🛠 Major Engineering Challenges & Solutions

### 🚨 Issue: Aggregation Returning `NaN`

Problem:

```cypher
SUM(t.price * t.quantity) → NaN
```

Root Cause:

* 600+ transactions contained floating-point `NaN`
* Neo4j propagates NaN through arithmetic
* `coalesce()` does NOT fix NaN

Solution:

```cypher
MATCH (t:Transaction)
WHERE t.price <> t.price
SET t.price = NULL
```

Cleaned:

* 622 `price` values
* 644 `quantity` values

Result:
Accurate aggregation restored.

---

### 🚨 Issue: Duplicate Review Results

Cause:
Multiple matching traversal paths.

Solution:
Used `DISTINCT` in Cypher queries.

---

### 🚨 Issue: Category Not Enforced in Hybrid Query

Solution:
Added entity detection:

```python
categories = ["furniture", "electronics", ...]
```

And enforced:

```cypher
AND ($category IS NULL OR p.product_category = $category)
```

---

## 📊 Example Outputs

### Top Spending Customers

```json
[
  {"customer": "Tracey Patterson", "total_spent": 145080.34},
  {"customer": "Margaret Liu", "total_spent": 116546.58}
]
```

---

### Hybrid Query Example

```python
ask_graph("Show durable furniture reviews from California")
```

Output:

```json
[
  {
    "review": "Perfect Fit for Our Living Room!",
    "product": "Sofa",
    "customer": "David Roberts",
    "score": 0.746,
    "rating": 5
  }
]
```

---

## 📈 Project Impact

* Modeled 30K+ relationships in Neo4j
* Implemented 384-dimension vector index
* Built hybrid GraphRAG pipeline
* Resolved NaN propagation affecting financial analytics
* Combined semantic retrieval with structured constraints

---

## 🛠 Tech Stack

* Python
* Neo4j AuraDB (v5)
* Cypher
* Sentence Transformers
* Pandas
* Google Colab

---

## 🧩 Future Improvements

* Replace rule-based router with LLM-based NL → Cypher
* Add conversational memory
* Deploy as Streamlit web application
* Add recommendation engine
* Implement evaluation metrics (precision@k)

---

## 🏆 What This Project Demonstrates

* Graph data modeling
* Cypher query optimization
* Vector search integration
* Hybrid semantic + symbolic retrieval
* Data cleaning & debugging (NaN handling)
* Production-style GraphRAG architecture

---

## 📌 Author

Built as an advanced GraphRAG engineering project exploring hybrid retrieval systems using Neo4j and vector embeddings.

---

