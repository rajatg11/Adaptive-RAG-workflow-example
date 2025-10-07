# Adaptive-RAG-workflow-example

This project will cover a full hands-on workflow and demonstration of how to build an Agentic Adaptive RAG System with LangGraph

The idea would be to implement the workflow taking inspiration from the [Adaptive-RAG](https://arxiv.org/pdf/2403.14403) research paper.

The main challenge of RAG systems include:

- Poor Retrieval can lead to issues in LLM response generation
- Bad retrieval or lack of information in the vector database can also lead to out of context or hallucinated answers
- Even complex RAG Systems like CRAG suffer from the fact that they are fixed flows which will always be executed regardless of query complexity

The idea is to build a dynamic agentic RAG system which can adapt based on the input query and route it to the best possible RAG workflow to handle it. In our case we will take a user query and route it either to be handled by the:
- Web Search based RAG Flow
- Vector DB based RAG flow

We will also add in elements of Corrective RAG and Self-Reflective RAG here.

We can build this as an agentic RAG system by having a specific functionality step as a node in the graph and use LangGraph to implement it. Key steps in the node will include prompts being sent to LLMs, tools, DBs to perform specific tasks as seen in the detailed workflow below:

![](https://i.imgur.com/ESG2Jc7.png)



### Adaptive RAG System with LangGraph

This project implements an **Adaptive RAG System** using LangGraph, designed to dynamically route user queries through the most suitable Retrieval-Augmented Generation (RAG) workflow. It supports both **Vector DB-based** and **Web Search-based** retrieval paths and incorporates elements from **Corrective RAG** (document grading) and **Self-Reflective RAG** (hallucination detection and feedback correction).

The adaptive workflow includes the following key components:

1. **Dynamic Query Routing**:
   - A **Query Router Prompt** classifies the user query and decides whether the information should be retrieved from:
     - An internal **Vector Database**, or
     - A **Web Search** engine.
   - Based on the routing decision, the query is rephrased using:
     - **VectorDB Rephrase Prompt** for vector-optimized semantic search, or
     - **Web Search Rephrase Prompt** for better recall from web sources.

2. **Retrieval and Context Preparation**:
   - **If the source is Vector DB**:
     - Rephrased query is used to retrieve documents from the **Vector Database**.
     - Retrieved documents are passed through an **LLM Grader Prompt**, which filters relevant documents using semantic grading (`yes` or `no`).
     - If **more than 50%** of documents are relevant, they are forwarded to the next stage.
     - If **50% or fewer** documents are relevant, the system switches to the **Web Search** path.
     - Any irrelevant documents are removed.

   - **If the source is Web Search** (or >=50% docs irrelevant from Vector DB route):
     - The rephrased query is used directly to search the web and retrieve external context documents.

3. **Answer Generation (RAG Prompt)**:
   - Final relevant documents — either from the Vector DB (filtered) and / or Web Search (raw) — are used in the **RAG Prompt**.
   - The prompt instructs the LLM to answer only using the provided context.
   - If feedback exists (from a prior iteration in case there were hallucinations), it is also included in the prompt to improve the response.

4. **Self-Reflective Hallucination Check**:
   - The generated answer undergoes evaluation using a **Hallucination Check Prompt**.
   - The LLM checks if the answer is grounded in the context documents and returns:
     - A **hallucination flag** (`yes` or `no`)
     - Optional **feedback** explaining the judgment.

5. **Feedback Loop for Regeneration**:
   - If the hallucination flag is `no`, the answer is finalized.
   - If it is `yes`, the feedback is used to regenerate the response using the same context via the **RAG Prompt**, making the system more reliable and self-correcting.

This **Adaptive RAG System** combines adaptive routing, corrective RAG, and self-reflective RAG elements to produce accurate, high-quality answers.
