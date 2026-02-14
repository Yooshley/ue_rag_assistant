# Unreal Engine Source-Grounded RAG

A **source-grounded Retrieval-Augmented Generation (RAG) system** for Unreal Engine C++ that answers engine behavior questions by retrieving and citing **exact Unreal Engine source code**, rather than relying on documentation, forum posts, or prior model knowledge.

The system is designed for **correctness, traceability, and debugging**.

---

## Motivation

Large language models often struggle with Unreal Engine because:
- Souce code gets deprecated, shifted, or rewritten across different versions
- Online discussions on forums may span multiple engine versions
- Unreal's source code is better documented and always up-to-date via developer comments

As a result, generic LLM responses can be incorrect even when they sound plausible.

This system avoids those issues by using **version-specific engine source code** as the **Knowledge Base** for retrieval, enabling queries such as:
- Why does `FGameplayEffectContextHandle` not replicate `SourceObject`?
- What is the difference between `FGameplayEffectContext` and `FGameplayEffectContextHandle`?
- What conditions gate specific ability behavior?

All responses are grounded in the retrieved engine source code and include verifiable citations.


---

## Design Principles

- Unreal Engine source code is the primary source of truth  
- No speculation or undocumented assumptions
- Every claim must be cited and verifiable in engine source
- Refuse to answer if grounding is insufficient

---

## Technologies Used

- Python
- LangChain
- ChromaDB
- HuggingFace embeddings (`bge-base-en-v1.5`)
- Ollama (locally inferencing `llama3.1:8b`)
- Unreal Engine 5.7

---

## What the System Does **CURRENTLY**

### Engine-aware ingestion
- Reads Unreal Engine source directly from an installation location
- Automatically detects engine version (currently UE 5.7)
- Focuses on the GameplayAbilities plugin (GAS) (will expand in future)

### Symbol-level chunking
- Extracts complete C++ symbols (types and functions)
- Uses custom braces based parsing for C++ code, avoiding issues with LLM chunking
- Preserves exact file paths and line ranges for precise citation

### Dense retrieval over engine code
- Uses `bge-base-en-v1.5` embeddings optimized for retrieval
- Stores vectors in a persistent Chroma database
- Retrieves symbols based on semantic intent

### Deterministic deduplication
- Collapses multiple hits into one result per symbol
- Prevents repeated explanations of the same implementation

### Strict citation enforcement
- Every claim must include a source citation
- Answers are rejected if citations are malformed or unsupported

### Offline, local inference
- Uses Ollama for API-free, local inference (Currently using llama3.1:8b)
- Can be run offline after ingestion

---

## Supported Question Types **AS OF NOW**

### Supported
- Behavioral / causal questions  
- Why does X happen?
- Why does Y not replicate?
- Where is Z implemented?

These are answered using `.cpp` implementations.

### Unsupported
- Definition questions (e.g. “What is `FGameplayEffectContext`?”)
- Function signature questions (e.g. “What arguments does this take?”)

These are planned for future versions.

---

## What Makes This Different From “Just Asking an LLM”

- The model cannot answer without retrieved source
- The model cannot invent explanations
- The model cannot generalize beyond cited code
- Answers can be verified directly in an IDE
