# Modern Databases: EPL 2024/25 Graph vs Document Database Comparison

A hands-on comparison of two NoSQL database types, **Neo4j (graph database)** and **MongoDB (document database)**, using the same real dataset: English Premier League 2024/25 season match data. Done for my Modern Databases module.

## The Brief

Model the same football dataset (teams, matches, referees, goals, results) in both a graph database and a document database, then write queries in each to answer the same set of questions (all the EPL teams, the highest scoring team, the final league table, the referees who took the most matches, and the longest winning streak after Christmas), then compare how each database handled the same problem.

## Why This Dataset Suits Both

- Teams are linked to each other through the matches they play, and referees are linked to the matches they officiate, which is a natural fit for a **graph** model (`Team`/`Match`/`Referee` nodes joined by `PLAYED_HOME`, `PLAYED_AWAY`, `OFFICIATED` relationships).
- Each match is also a self-contained record on its own (two teams, goals, result, referee, venue), which suits a **document** model well, where one MongoDB document just holds everything about that one match.

## Neo4j (Graph Database)

Modelled using `Team`, `Match` and `Referee` nodes connected by relationships. Example query (all EPL teams):

```cypher
MATCH (t:Team)
RETURN t.name
ORDER BY t.name;
```

Example (total goals scored/conceded at home, added up across connected matches):

```cypher
MATCH (t:Team)-[:PLAYED_HOME]->(m:Match)
WITH t,
     SUM(m.homeGoals) AS GoalsScored,
     SUM(m.awayGoals) AS GoalsConceded
RETURN t.name AS Team, GoalsScored, GoalsConceded
ORDER BY GoalsScored DESC;
```

## MongoDB (Document Database)

Modelled as one `epl_matches` collection, with every match stored as a single document. Example query (all EPL teams):

```javascript
db.epl_matches.distinct("home")
```

Example (total goals scored per team, using an aggregation pipeline):

```javascript
db.epl_matches.aggregate([
  { $group: { _id: "$home", TotalGoals: { $sum: "$goals_home" } } },
  { $sort: { TotalGoals: -1 } }
])
```

## Comparing the Two

| | Neo4j (Graph) | MongoDB (Document) |
|---|---|---|
| **Good at** | Queries about relationships (team ↔ match ↔ referee) | Self-contained records, stats-heavy queries |
| **Query style** | Cypher, pattern matching over nodes/relationships | Aggregation pipelines |
| **Flexibility** | Great for connected data, less natural for big flat document storage | Easy to add new fields later (e.g. yellow cards, possession %) without changing the structure |
| **Struggled with** | Handling large amounts of unstructured document-style data | Relationship queries needed longer, more complex pipelines |

**What I took from it:** there isn't really a "better" database overall, since it depends on whether your queries care more about relationships (graph) or about whole records (document). Neo4j made relationship-based questions, like which referee did which matches, genuinely easy to write. MongoDB made totals and groupings quick and straightforward, since each match's full details were already sitting in one place.

## Tech Used

Neo4j, Cypher, MongoDB, MongoDB Aggregation Framework

## Note

This was university coursework using a real, publicly available EPL 2024/25 dataset, not a production system.
