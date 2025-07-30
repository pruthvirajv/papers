# OntoLLM Prompt Templates

This directory contains the prompt templates used by the OntoLLM framework for various language model interactions. These prompts are designed to enable effective ontology-guided query generation, context management, and information retrieval in industrial conversational AI applications.

## Overview

The OntoLLM framework uses structured prompts to guide large language models in generating accurate queries, rewriting user inputs for better context, and retrieving relevant information from knowledge graphs and vector databases. Each prompt template is carefully crafted to incorporate domain-specific ontological knowledge and maintain conversational coherence.

## Prompt Files

### 1. Query Rewrite Prompt (`QueryReWrite.prompt`)

**Purpose**: Enhances user queries by incorporating conversation context and resolving coreferences.

**Use Case**: Pre-processing step in the OntoLLM pipeline to ensure contextually complete queries before retrieval.


#### Key Features:
- **Context Integration**: Incorporates previous conversation turns
- **Coreference Resolution**: Resolves pronouns and implicit references
- **Query Completion**: Generates self-contained queries for better retrieval
- **Domain Awareness**: Uses industrial examples (motor models, policies)

#### Algorithm Reference:
Used in Algorithm `QueryRewriting` (Algorithm 1) in the OntoLLM paper.

---

### 2. Query Prompt Construction (`QueryPromptConstruction.prompt`)

**Purpose**: Generates structured knowledge graph queries using ontological examples and domain-specific patterns.

**Use Case**: Core component of the Structured Query Handler (SQH) for generating precise SPARQL/Cypher queries.


#### Key Features:
- **Ontology-Grounded**: Uses complete ontology structure as context
- **Few-Shot Learning**: Provides natural language to KG query examples
- **Entity Parameterization**: Uses `#ent1`, `#ent2` placeholders for entity values
- **Neo4j Compatibility**: Generates Cypher queries for Neo4j graph database

#### Algorithm Reference:
Used in Algorithm `CreateQueryPrompt` (Algorithm 6) in the OntoLLM paper.

---

### 3. Vector Reference Generation (`VectorRefGeneration.prompt`)

**Purpose**: Generates knowledge graph queries specifically designed to retrieve vector database references for unstructured content retrieval.

**Use Case**: Core component of the Unstructured Query Handler (UQH) for bridging structured knowledge with unstructured documents.


#### Key Features:
- **Vector Bridge Focus**: Specifically targets Question nodes containing `ChunkVectorID`
- **Context-Driven**: Uses entity context to filter relevant vector references
- **Chain Navigation**: Follows ontological relationships to reach Question nodes
- **Reference Retrieval**: Returns vector database identifiers for precise chunk retrieval

#### Algorithm Reference:
Used in the Unstructured Query Handler section for vector reference generation.



