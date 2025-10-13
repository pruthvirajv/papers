# Amazon Ontology

This directory contains `amazon_ontology.json`, a small ontology describing the schema and relationships for an Amazon-like product dataset used for recommendation and similarity tasks.

Contents
- `amazon_ontology.json` — JSON file describing nodes, relation paths and chainRank configurations.

Schema summary
- Nodes:
  - Product: { ProductID: Numeric, ProductName: Text, Category: Text, Price: Numeric, Brand: Text }
  - User: { UserID: Numeric, AgeGroup: Text, Gender: Text, Occupation: Text }
  - Rating: { UserID: Numeric, ProductID: Numeric, Rating: Numeric, Timestamp: Numeric }
  - Category: { CategoryName: Text }
  - Brand: { BrandName: Text }

Relations (relation trees)
- Paths listed in `relationtrees.paths` (example):
  - (P:Product)-[hc:hasCategory]-(C:Category)
  - (P:Product)-[hb:hasBrand]-(B:Brand)
  - (U:User)-[r:RATED]-(P:Product)

chainRank
- `chainRank` defines ranking or similarity chain configurations used by downstream algorithms. Example keys:
  - UserProductSimilarity: evaluates similarity between `User` and `Product` using intermediate nodes like `Category` and `Brand` and relations such as `RATED`.
  - ProductSimilarity: compares `Product` to `Product` via shared `Category` and `Brand`.

Example usage
- Load the JSON and programmatically build a graph where nodes above become node types and paths define allowed traversals for feature extraction, path-based embeddings, or chain-ranking algorithms.

Minimal Python snippet
```
import json
from pathlib import Path

path = Path(__file__).with_name('amazon_ontology.json')
ontology = json.loads(path.read_text())
print(ontology['nodes'].keys())
```

Notes
- Field types are descriptive only (e.g., "Text", "Numeric"). Map them to concrete types in your processing pipeline.
- Adjust `relationtrees.paths` syntax to match the graph library you use (e.g., NetworkX, neo4j, DGL).
