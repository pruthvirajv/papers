# Movies Ontology

This directory contains `movie_ontology.json`, an ontology describing a movie-rating dataset (MovieLens-like). Use it for building recommendation models, graph features, and similarity metrics.

Contents
- `movie_ontology.json` — JSON file describing node types, relation trees and chainRank setups.

Schema summary
- Nodes:
  - Movie: { MovieID: Numeric, Title: Text }
  - User: { UserID: Numeric, Gender: Text, Zip-code: Text }
  - Rating: { UserID: Numeric, MovieID: Numeric, Rating: Numeric, Timestamp: Numeric }
  - Genres: { GenresName: Text }
  - ReleaseYear: { ReleaseYearValue: Numeric }
  - Age: { AgeID: Numeric, AgeCategory: Text }
  - Occupation: { OccupationID: Numeric, OccupationCategory: Text }
  - Gender: { GenderID: Text, GenderName: Text }

Relations (relation trees)
- Example paths:
  - (MO:Movie)-[hg:hasGenres]-(G:Genres)
  - (MO:Movie)-[hr:hasReleaseYear]-(R:ReleaseYear)
  - (U:User)-[ha:hasAge]-(A:Age)
  - (U:User)-[ha:hasGender]-(G:Gender)
  - (U:User)-[hg:hasGenres]-(G:Genres)
  - (U:User)-[ho:hasOccupation]-(O:Occupation)
  - (U:User)-[r:RATED]-(M:Movie)

chainRank
- Predefined ranking configurations:
  - UserMovieSimilarity: connect `User` to `Movie` via `Genres`.
  - MovieSimilarity: `Movie` to `Movie` via `Genres` and `ReleaseYear`.
  - UserSimilarity: `User` to `User` via `Genres`, `Age`, `Occupation` (and `Gender`).

Example usage
- Use the ontology to create graph-based features for collaborative filtering and content-aware recommendation models. Paths map naturally to meta-path based similarity measures.

Minimal Python snippet
```
import json
from pathlib import Path

path = Path(__file__).with_name('movie_ontology.json')
ontology = json.loads(path.read_text())
print('Defined chainRank entries:', list(ontology.get('chainRank', {}).keys()))
```

Notes
- The relation and node labels are illustrative — adapt them to your dataset's naming conventions before ingestion.
