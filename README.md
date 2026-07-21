# Modern Databases — EPL 2024/25 Graph vs Document Database Comparison

A hands-on comparison of two NoSQL database paradigms — **Neo4j (graph database)** and **MongoDB (document database)** — modelling and querying the same real-world dataset: English Premier League 2024/25 season match data. Built for my Modern Databases module.

## The Brief

Model the same football dataset (teams, matches, referees, goals, results) in both a graph database and a document database, then write queries to answer the same set of analytical questions in each — identifying all EPL teams, the highest-scoring team, the final league table, referees with the most appearances, and the longest post-Christmas winning streak — before comparing how each database approached the same problem.

## Why This Dataset Fits Both Models

- Teams are connected to each other through the matches they play, and referees are connected to the matches they officiate — a natural fit for a **graph** model (`Team`/`Match`/`Referee` nodes linked by `PLAYED_HOME`, `PLAYED_AWAY`, `OFFICIATED` relationships)
- Each match is also a fully self-contained record (two teams, goals, result, referee, venue) — a natural fit for a **document** model, where one MongoDB document holds everything about a single match

## Neo4j (Graph Database)

Modelled with `Team`, `Match`, and `Referee` nodes connected by relationships. Example query — all EPL teams:

```cypher
MATCH (t:Team)
RETURN t.name
ORDER BY t.name;
```

Example — total goals scored/conceded at home, aggregated across connected matches:

```cypher
MATCH (t:Team)-[:PLAYED_HOME]->(m:Match)
WITH t,
     SUM(m.homeGoals) AS GoalsScored,
     SUM(m.awayGoals) AS GoalsConceded
RETURN t.name AS Team, GoalsScored, GoalsConceded
ORDER BY GoalsScored DESC;
```

## MongoDB (Document Database)

Modelled as a single `epl_matches` collection, with each match stored as one document. Example query — all EPL teams:

```javascript
db.epl_matches.distinct("home")
```

Example — total goals scored per team, using an aggregation pipeline:

```javascript
db.epl_matches.aggregate([
  { $group: { _id: "$home", TotalGoals: { $sum: "$goals_home" } } },
  { $sort: { TotalGoals: -1 } }
])
```

## Comparison: Graph vs Document

| | Neo4j (Graph) | MongoDB (Document) |
|---|---|---|
| **Best at** | Relationship-heavy queries (team ↔ match ↔ referee connections) | Self-contained records, aggregation-heavy stats |
| **Query style** | Cypher — pattern matching over nodes/relationships | Aggregation pipelines |
| **Flexibility** | Strong for connected data; less natural for large flat document storage | Easy to add new fields (e.g. yellow cards, possession %) without restructuring |
| **Where it struggled** | Less convenient for large, unstructured document-style data | Relationship-style queries needed more complex, longer pipelines |

**Takeaway:** there's no universally "better" database — the right choice depends on whether your queries are relationship-driven (→ graph) or record-driven (→ document). Neo4j made relationship queries (e.g. referee involvement across matches) genuinely simple to express; MongoDB made aggregate statistics (totals, counts, groupings) fast and natural, since each match's full context already lived in one document.

## Tech Used

Neo4j, Cypher, MongoDB, MongoDB Aggregation Framework

## Note

This was university coursework analysing a real, publicly available EPL 2024/25 dataset — not a production system.
