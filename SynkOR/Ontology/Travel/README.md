# Travel Ontology

This directory contains `travel_ontology.json`, an ontology modelling destinations, users, ratings and related types for travel recommendation tasks.

Contents
- `travel_ontology.json` — JSON file listing node types, relation paths and chainRank configurations.

Schema summary
- Nodes:
  - Destination: { DestinationID: Numeric, Name: Text, Country: Text, Popularity: Numeric, PriceLevel: Numeric }
  - User: { UserID: Numeric, AgeGroup: Text, TravelPreference: Text, TravelStyle: Text, PreferredAccommodation: Text }
  - Rating: { UserID: Numeric, DestinationID: Numeric, Rating: Numeric, Timestamp: Numeric }
  - Type: { TypeName: Text }
  - Activity: { ActivityName: Text }
  - Accommodation: { AccommodationName: Text }

Relations (relation trees)
- Example paths:
  - (D:Destination)-[ht:hasType]-(T:Type)
  - (D:Destination)-[ha:hasActivity]-(A:Activity)
  - (D:Destination)-[hc:hasAccommodation]-(C:Accommodation)
  - (U:User)-[r:RATED]-(D:Destination)
  - (U:User)-[hp:hasPreferredAccommodation]-(C:Accommodation)

chainRank
- Configurations include:
  - UserDestinationSimilarity: compares `User` to `Destination` using Type/Activity/Accommodation and `RATED` relations.
  - DestinationSimilarity: compares `Destination` to `Destination` via shared Type/Activity/Accommodation.

Example usage
- Use this ontology to derive meta-paths for preference-aware recommendations, activity-based similarity, or to construct graphs for GNN-based ranking.

Minimal Python snippet
```
import json
from pathlib import Path

path = Path(__file__).with_name('travel_ontology.json')
ontology = json.loads(path.read_text())
print('Nodes:', ', '.join(ontology['nodes'].keys()))
```

Notes
- Map the generic field types to concrete dataset types when loading data. The paths are example-style and may need small edits when used with a specific graph database.
# Travel Recommendation Dataset

## Overview

The **Travel Recommendation Dataset** is a comprehensive dataset designed for building and evaluating conversational recommendation systems in the travel domain. It includes detailed information about users, destinations, and ratings, enabling researchers and developers to create personalized travel recommendation models.

---

## Dataset Contents

The dataset consists of three JSON files:

### 1. `users.json`
- **Description**: Contains user profiles with demographic information and travel preferences.
- **Fields**:
  - `userId`: Unique identifier for each user.
  - `age_group`: Age group of the user (e.g., "18-24", "25-34").
  - `travel_preference`: Budget preference (e.g., "Budget", "Mid-range", "Luxury").
  - `travel_style`: Travel style (e.g., "Solo", "Couple", "Family", "Friends").
  - `preferred_activities`: List of activities the user prefers (e.g., "Sightseeing", "Museum visits").
  - `preferred_accommodation`: Preferred type of accommodation (e.g., "Hotel", "Camping").

### 2. `destinations.json`
- **Description**: Contains information about travel destinations, including their attributes and popularity.
- **Fields**:
  - `destinationId`: Unique identifier for each destination.
  - `name`: Name of the destination.
  - `country`: Country where the destination is located.
  - `types`: Types of the destination (e.g., "Adventure", "Relaxation").
  - `activities`: Activities available at the destination (e.g., "Surfing", "Hiking").
  - `accommodations`: Types of accommodations available (e.g., "Villa", "Hostel").
  - `popularity`: Popularity score (0-1 scale).
  - `price_level`: Price level (e.g., "Budget", "Mid-range", "Luxury").

### 3. `ratings.json`
- **Description**: Contains user ratings for destinations.
- **Fields**:
  - `userId`: ID of the user who rated the destination.
  - `destinationId`: ID of the rated destination.
  - `rating`: Rating given by the user (e.g., 1-5 scale).

---

## Use Cases

This dataset can be used for:
- Building conversational recommendation systems.
- Personalizing travel recommendations based on user preferences.
- Analyzing user behavior and travel trends.
- Training machine learning models for recommendation tasks.

---

## Dataset Statistics

- **Users**: 436 profiles.
- **Destinations**: 346 destinations.
- **Ratings**: (Add the number of ratings from `ratings.json`).

---

## Format

All files are in JSON format, making them easy to parse and integrate into various machine learning pipelines.

---


