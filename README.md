NEURAL MEMORY — Index 12 Retrieval

Semantic Retrieval for large codebases — from user intent to relevant code context.

Index 12 extends NEURAL MEMORY from a semantic/project visualization into a usable retrieval system for large software projects.

The core idea is simple:

Instead of repeatedly giving an LLM thousands of lines of source code, retrieve the small set of code regions that are semantically relevant to the current task.

The system separates semantic indexing, retrieval, graph structure and visual inspection.

⸻

What problem does it solve?

Large software projects quickly become difficult for an LLM to work with.

A typical project contains:

* many files
* classes and functions
* UI components
* configuration
* dependencies
* historical changes
* duplicated concepts
* code that is relevant only indirectly

Sending the complete project to an LLM is expensive and produces unnecessary context.

NEURAL MEMORY creates a persistent semantic representation of the project.

A query can then activate the relevant parts of that representation.

User Query
    │
    ▼
Embedding
    │
    ▼
Semantic Retrieval
    │
    ▼
Top-K relevant Nodes
    │
    ▼
Related Code / Symbols
    │
    ▼
Context Builder
    │
    ▼
LLM

The LLM receives selected context, rather than the entire project.

⸻

Architecture

                ┌──────────────────────┐
                │      Source Code     │
                └──────────┬───────────┘
                           │
                           ▼
                ┌──────────────────────┐
                │  Project Indexing    │
                │                      │
                │ files / symbols      │
                │ blocks / relations   │
                └──────────┬───────────┘
                           │
                           ▼
                ┌──────────────────────┐
                │ Embedding Model      │
                │                      │
                │ multilingual-e5      │
                │ 384 dimensions       │
                └──────────┬───────────┘
                           │
                           ▼
                ┌──────────────────────┐
                │ Semantic Memory      │
                │                      │
                │ vectors + metadata   │
                │ graph relations      │
                └──────────┬───────────┘
                           │
                    semantic query
                           │
                           ▼
                ┌──────────────────────┐
                │ Deterministic        │
                │ Retrieval            │
                │                      │
                │ cosine similarity    │
                │ Top-K selection      │
                └──────────┬───────────┘
                           │
                           ▼
                ┌──────────────────────┐
                │ Context Builder      │
                └──────────┬───────────┘
                           │
                           ▼
                         LLM

The important boundary is:

Retrieval determines what is relevant.
The LLM operates only on the resulting context.

⸻

Index 12

Index 12 is the current retrieval-oriented stage of NEURAL MEMORY.

Earlier versions established the semantic memory representation and its visualization.

Index 12 adds the missing bridge:

Semantic Memory
       ↓
      Query
       ↓
Similarity Search
       ↓
Top-K Retrieval
       ↓
Relevant Project Context

This turns the semantic graph into something that can be used operationally.

⸻

Semantic Representation

Each indexed unit receives an embedding.

The current embedding space uses:

Xenova/multilingual-e5-small

with:

dimension = 384

Each node therefore contains a complete semantic vector:

vector ∈ R³⁸⁴

The 3D representation shown in the UI is not the semantic representation.

It is only a visualization.

384D vector
    │
    │ semantic truth
    ▼
cosine similarity

versus:

3D position
    │
    │ visualization
    ▼
human inspection

This distinction is fundamental.

384D is used for retrieval. 3D is used for understanding.

⸻

Retrieval

A query is converted into the same embedding space as the indexed nodes.

For a query vector q and node vector v, semantic similarity is calculated using cosine similarity:

[
sim(q,v)=
\frac{q\cdot v}
{|q||v|}
]

The nodes are then ordered according to their similarity to the query.

For example:

Query:
"Which components control the smartphone UI layout?"
            ↓
Embedding
            ↓
Cosine Similarity
            ↓
┌──────────────────────────────┐
│ 1. UI layout component       │
│ 2. responsive renderer       │
│ 3. viewport configuration    │
│ 4. CSS / layout block        │
│ 5. interaction state         │
│ ...                          │
└──────────────────────────────┘
            ↓
Top-K context

The retrieval operation itself does not require an LLM.

⸻

Top-K Retrieval

The system can restrict the result to the most relevant nodes.

Conceptually:

TopK(q, V, k)

where:

* q = query vector
* V = indexed node vectors
* k = number of results

Example:

K = 8

The result is therefore a bounded context instead of the entire project.

This is especially important for large codebases.

⸻

Code Structure

The indexed project is not treated as one large text blob.

The system extracts structural units such as:

Project
 ├── File
 │    ├── Symbol
 │    ├── Function
 │    ├── Class
 │    └── Code Block
 │
 └── Relations
      ├── contains_symbol
      ├── related_to
      └── semantic_neighbor

This allows retrieval to operate on meaningful project elements.

A retrieved semantic node can therefore be connected back to the corresponding project structure.

⸻

Graph + Semantic Space

NEURAL MEMORY combines two different structures.

Semantic layer

Answers:

What is conceptually similar?

Embedding
   ↓
384D vector
   ↓
Cosine similarity

Structural layer

Answers:

What belongs to what?

File
 └── Symbol
      └── Code Block

Both can be combined during context construction.

Query
  │
  ▼
Semantic Retrieval
  │
  ├── Node A
  ├── Node B
  └── Node C
       │
       ▼
Structural Relations
       │
       ▼
Relevant source regions

This is more useful than pure vector search alone because semantic relevance can be connected back to the actual project structure.

⸻

Visual Retrieval

The 3D interface provides a human-readable representation of the retrieval space.

Nodes can be highlighted according to their relationship to the current query.

This makes it possible to inspect:

Query
  │
  ├── highly relevant nodes
  ├── neighboring concepts
  ├── structural relations
  └── connected project regions

The visualization is therefore not the retrieval algorithm.

It is an inspection interface for the retrieval system.

⸻

Example: Smartphone UI

Suppose the project contains a large UI.

The user asks:

"Why do I only see one third of the interface
on a smartphone?"

The retrieval layer searches the semantic project representation.

Potentially relevant regions include:

viewport
responsive layout
camera
Three.js renderer
canvas dimensions
CSS positioning
UI overlay
mobile scaling

Instead of loading the entire project, the system can identify the most relevant nodes first.

The next step is then to resolve those nodes to their actual source-code regions.

Query
  ↓
Top-K semantic nodes
  ↓
Source symbols / blocks
  ↓
Relevant code
  ↓
Context Builder
  ↓
LLM

This is the intended use of NEURAL MEMORY.

⸻

Why the 3D Graph Matters

The graph is not intended to replace retrieval.

It provides a way to see what retrieval means inside the project.

For example:

             semantic neighborhood
                   ●
              ●         ●
                  ●
          ●       QUERY       ●
                  ●
             ●         ●
                   ●

A retrieved node can then be traced through its graph relations to other project elements.

This gives the developer a visual explanation of the retrieved context.

⸻

Determinism

The retrieval pipeline is intentionally separated from generative reasoning.

The system can therefore reproduce a retrieval operation from:

project version
+
embedding model version
+
embedding vectors
+
retrieval parameters
+
query

For a fixed indexed state and fixed parameters, the retrieval operation is deterministic.

This is important for debugging and evaluation.

⸻

Current Parameters

The semantic memory architecture uses parameters such as:

Embedding dimension: 384
Model: multilingual-e5-small
Top-K: 8
Edge threshold: 0.72
Neighbor count: 8
Anchor count: 5

These parameters belong to the system configuration rather than being invented by the LLM.

⸻

LLM Boundary

The LLM is deliberately not responsible for executing retrieval logic.

The conceptual architecture is:

                 LLM
                  │
            query / task
                  │
                  ▼
        ┌─────────────────┐
        │ Retrieval Layer │
        └────────┬────────┘
                 │
             Top-K nodes
                 │
                 ▼
        ┌─────────────────┐
        │ Context Builder │
        └────────┬────────┘
                 │
                 ▼
                 LLM

The model can formulate or interpret a task.

The deterministic system decides which indexed artifacts correspond to the retrieval operation.

This keeps the system boundary explicit.

⸻

From Retrieval to Project Memory

Index 12 is one step toward a larger architecture:

                 PROJECT
                    │
          ┌─────────┴─────────┐
          │                   │
     Project State        Semantic Memory
          │                   │
     Git / commits        embeddings
     diffs / patches      concepts
     code state           relations
          │                   │
          └─────────┬─────────┘
                    │
                    ▼
              Context Builder
                    │
                    ▼
                   LLM

The project therefore becomes more than a collection of files.

It becomes a persistent, queryable memory representation.

⸻

Design Principles

1. Semantic truth ≠ visualization

The 384-dimensional embedding is the semantic representation.

The 3D graph is only a visualization.

⸻

2. Retrieval ≠ generation

Retrieval identifies relevant project context.

The LLM generates an interpretation or proposal from that context.

⸻

3. Structure matters

A semantic match should remain connected to the actual project structure.

⸻

4. No fake project data

The graph is intended to represent actual indexed project artifacts.

The visualization should emerge from the project itself.

⸻

5. Reproducibility

The retrieval result should be explainable through:

query
+
embedding
+
similarity
+
parameters
+
project state

⸻

Roadmap

Current

* Project indexing
* Semantic embeddings
* 384D vector representation
* Cosine similarity
* Semantic neighbors
* Top-K retrieval foundation
* 3D semantic visualization
* Structural project nodes

Next

* Explicit retrieval benchmark
* Query → Top-K evaluation dataset
* Precision@K / Recall@K measurements
* Retrieval → source-code resolution
* Context Builder
* UI-specific retrieval
* Project-state aware retrieval
* Git commit / diff integration
* Retrieval explanations
* LLM integration behind a strict context boundary

⸻

The Core Idea

NEURAL MEMORY is based on a simple observation:

An LLM does not need to remember an entire project if the system can deterministically retrieve the part of the project that matters.

Index 12 explores that layer.

The goal is not to create a prettier code graph.

The goal is to create a persistent semantic index of a software project that can be queried, inspected and used to construct precise LLM context.

              LARGE PROJECT
                    │
                    ▼
             NEURAL MEMORY
                    │
             semantic index
                    │
                    ▼
                 QUERY
                    │
                    ▼
              TOP-K RETRIEVAL
                    │
                    ▼
             RELEVANT CONTEXT
                    │
                    ▼
                   LLM

NEURAL MEMORY — making project context retrievable.