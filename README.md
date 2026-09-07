# Neo4j Synthetic Transaction Graph — Fraud Sandbox

Generates a synthetic financial-transaction network with **Faker** and loads it into **Neo4j** — customers as nodes, money transfers as relationships — to build and test fraud-detection scenarios on realistic-looking data without touching real records. Built during solo fraud-detection R&D at DataSpecta.

## How it works

1. **Generate customers & transfers** — `Faker` creates sender IDs (`FRM-1#####`), receiver IDs (`TO-1#####`), transfer amounts (2.00–10,000.00) and timestamps.
2. **Build the network** — each sender gets 1–10 randomized outgoing transfers to other customers; self-transfers are detected by ID comparison and skipped. Repeated transfers between the same pair are kept as amount/timestamp lists.
3. **Export** — the final DataFrame is written to CSV (a sample generated dataset, `benemre`, is included in the repo).
4. **Load into Neo4j** — via the Bolt driver (`neo4j://localhost:7687`): `Customer` nodes are created, transfers become `[:Transaction {amount, time}]` relationships between them; a `clear_db` helper wipes the graph between runs. The resulting graph is then explored/visualized in Neo4j to simulate fraud patterns.

## Stack

Python · pandas · Faker · Neo4j (Bolt driver) · Jupyter

## Run

```bash
pip install -r requirements.txt neo4j
```

1. Start a local Neo4j instance (default `neo4j://localhost:7687`) and set your credentials in the driver cell.
2. Open `fake_data.ipynb` and run top-to-bottom — the notebook asks for the row count interactively.
3. Inspect the graph in Neo4j Browser (`MATCH (c:Customer)-[t:Transaction]->(c2) RETURN c,t,c2 LIMIT 100`).
