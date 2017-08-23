# EToiLe
a declarative, lineage-centric ETL framework that enforces data engineer best practices

**EToiLe** is an opinionated ETL framework that enforces a set of
hard constraints and provides rich guarantees as
a results. Here are some of the properties of the framework:

* **declarative in nature:** but supports implementation in multiple execution engines
* **functional:** enforces functional programming concepts to batch processing.
  * **pure-tasks:** akin to pure functions as units of computation with no side-effects
  * **immutability:** of all data blocks if enforced
  * **idempotency:** all tasks use a set of blocks as input, and outputs one block,
    given the same task version and input blocks, will always give the same output block
* **metadata centric**:
  * **hollistic data lineage:** EToiLe is fully aware of column **and** block-level
    lineage information
  * **task instance metadata:** captures every task instance detail,
    with the exact version of logic that was executed, input and output blocks.
    While the DAG specification or shape changes over time, EToiLE keeps tab
    of everything that ever took place
  * **operational metadata:** captures compute and storage information, helping
    to inform capacity planning and predict backfill costs
  * **ownership aware** able to inform downstream crowds of upstream changes
* **metadata driven:**
  * **scheduling:** EToiLe works like a compiler. Given changes on logic
    or the mutation of data blocks, it knows exactly what needs to be [re]processed
  * **compute accumulation:** EToiLe knows and can report on which blocks
    are "dirty" (need to be recomputed) based on upstream changes. "Dirt" can
    be enumerated for each block, table, or subDAG, and can be accumulated over time
    until paying the [estimatable] cost of backfilling is triggered
  * **schema aware:** knowing your expectations around schemas, EToiLe can
    move your schema forward through dynamic DDL to do things like add new
    columns to your tables. When columns are removed on the logical layer,
    they are kept in the physical layer.
* **data aware:**
  * **block level statistics:** EToiLe, can compute statistics as it commits
  new blocks of data specified statistics about the data based on a stats
  rule engine. These statistics can be leveraged to enforce data quality
  checks and for anomaly detection
* **historical**: the full target state of the entire data warehouse is
  declared. The specification language allows for different logic to
  be applied for different, **non-overlapping** time periods

# Guarantees & potential
All of this metadata will make for a limitless UI.

* Navigate your current and historical DAGs of computation
* Navigate your table statistics
* Everything is fully reproduceable
* It's just one big DAG, no logical boundaries
* effortless to add columns

## Semantics & Concepts
* **dataset** is a table or collection of blocks that have a forward compatible schema
* **task** is a piece of logic that source from a collection of blocks and target a single block
* **run** is a single schedule and is time bound, it represents the left bound of the time interval of related task runs
* **task run** is a single run of a single task
* **subdag** is any subset of the graph
* **column set** is a set of columns that can be referenced as a group and evolve over time
  adding a new column to a set may be all you have to
* **metric expression** is the re

## Challenges
* heterogenous schedules, and depend-on-past-type constraints, the declarative
  language needs to be batch-aware

## Decisions to be made
* schedule centric?! enforce time-constrained schedules as opposed to batch
  ids
* use a graph database? GraphQL?

## Tangents
* Allow for specifying logical, dynamic named subdags. A logical subdag
  may be "all descendants of X", or "lineage between A and B", or
  "all of the ancestors of A and B"
* all dates are truncated on schedule
* targets can be virtual or materialized
* configs HAVE to be serializable so that they can stored in a database.
While the current target for the warehouse lives in source control, the history
of what ran is made available

#task
task_SHA
target (ix)
task_json

#task run
target(ix)
schedule_dttm(ix)
state [none, running, success, failure]
repo_SHA
task_SHA (fk)


```js
{
  target: "some_table",
  eql: "SELECT a, b, c FROM source_table",

  schedule: "1D",
  effective_date: "2016-01-01"

}
{
  target: "some_table",
  eql: "SELECT a, b, c FROM source_table",

  schedule: "1D",
  effective_date: "2017-01-01"
  end_date: "2017"
}
```
