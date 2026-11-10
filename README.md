# Synthetic transaction graph in Neo4j

Generates a synthetic network of money transfers with Faker and loads it into Neo4j, with customers as nodes and transfers as relationships. The goal was a sandbox for fraud-detection experiments that never touches real customer data. Built in 2022.

## Skills shown

- **Synthetic data generation:** realistic-looking IDs, amounts and timestamps with Faker, generated at whatever size the experiment needs.
- **Data wrangling in pandas:** reshaping generated records into a sender, receiver, amount, timestamp table and exporting it to CSV.
- **Data quality checks:** detecting self-transfers by comparing the numeric part of sender and receiver IDs.
- **Graph modelling:** mapping a tabular transfer log to a property graph (`(:Customer)-[:Transaction {amount, time}]->(:Customer)`).
- **Database loading:** Neo4j Bolt driver with parameterised Cypher queries (`$id`, `$amount`), which keeps generated values out of the query text.

## How it works

1. **Generate senders.** Faker creates sender IDs (`FRM-1#####`), amounts between 2.00 and 10,000.00 and timestamps from September 2022 onwards. The notebook asks for the number of rows.
2. **Generate receivers.** Sender IDs are relabelled to `TO-…` and shuffled, so every customer can appear on both sides.
3. **Check consistency.** Rows where a customer would send money to themselves are detected, and the fan-out step below skips them.
4. **Fan out transfers.** Each sender gets between 1 and 10 outgoing transfers to random receivers. Each sender–receiver pair carries two transfers, stored as lists of amounts and timestamps. (This step, in the cells marked with his name, was written by a colleague, Emre Kaya.)
5. **Export** the final table to CSV. `benemre` is a 500-row sample output.
6. **Load into Neo4j.** `clear_db` wipes the graph, `Customer` nodes are created and each transfer becomes a `Transaction` relationship.

## Run

1. Start Neo4j locally, for example: `docker run -p 7474:7474 -p 7687:7687 -e NEO4J_AUTH=neo4j/<password> neo4j:5`.
2. Put your password in the driver cell (`<your-neo4j-password>`).
3. `pip install -r requirements.txt neo4j`, open `fake_data.ipynb` and run it top to bottom.
4. Explore in Neo4j Browser at `http://localhost:7474`:

```cypher
MATCH (a:Customer)-[t:Transaction]->(b:Customer)
RETURN a, t, b LIMIT 100
```

## What I'd improve

- **Inject known fraud patterns as ground truth.** The current graph is random. The useful next step is planting patterns with labels, such as money cycles, fan-out to many new accounts, or one unusually large transfer from an account that normally sends small amounts, and then measuring whether a detector finds them.
- **No duplicate nodes.** The loader uses `CREATE`, so a receiver that appears twice becomes two nodes. `MERGE` plus a uniqueness constraint on `customer_id` fixes that.
- **Faster loading.** One transaction per row is slow for large graphs. Batching with `UNWIND` or `neo4j-admin database import` scales to millions of rows.
- **Reproducible runs.** A fixed random seed and command-line arguments instead of `input()` would make every dataset reproducible.
- **Credentials from the environment** instead of the notebook.
