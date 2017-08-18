# EToiLe
a declarative, lineage-centric ETL framework that enforces data engineer best practices

**EToiLe** is an opinionated ETL framework that enforces a set of
hard constraints and provides rich guarantees as
a results. Here are some of the properties of the framework:

* **declarative in naute:** but supports implementation in multiple execution engines
* **functional:** enforces functional programming concepts to batch processing.
  * **pure-tasks:** akin to pure functions as units of computation with no side-effects
  * **immutability:** of all data blocks if enforced
  * **idempotency:** all tasks use a set of blocks as input, and outputs one block,
    given the same task version and input blocks, will always give the same output block
* **metadata centric**:
  * **hollistic data lineage:** EToiLe is fully aware of column/block-level
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
    or the mutation data blocks, it knows exactly what needs to be processed
  * **compute accumulation:** EToiLe knows and can report on which blocks
    are "dirty" (need to be recomputed) based on upstream changes. "Dirt" can
    be enumerated for each block or subDAG, and can be accumulated over time
    until paying the [estimatable] cost of backfilling is triggered
  * **schema aware:** knowing your expectations around schemas, EToiLe can
    move your move forward through dynamic DDL to do things like add new
    columns to your tables
* **data aware:**
  * **block level statistics:** EToiLe, can compute statistics as it commits
  new blocks of data specified statistics about the data based on a stats
  rule engine. These statistics can be leveraged to enforce data quality
  checks and for anomaly detection

# Guarantees & potential
All of this metadata will make for a limitless UI.

* Navigate your current and historical DAGs of computation
* Navigate your table statistics
* Everything is fully reproduceable
* 


* It's just one big ass DAG
*
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
