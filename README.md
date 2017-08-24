# EToiLe
a declarative, data-lineage-centric ETL framework that enforces data engineering best practices

**EToiLe** is an opinionated ETL framework that enforces a set of
hard constraints and provides rich guarantees as a results.

Here are some of the properties of the framework:

* **declarative in nature:** EToiLe is a way to express large
  graphs of data structures and data transformations
* **compute translator** translations of the DSL into real execution plan
  on multiple platforms (Hive, Spark, Presto, ...) are in scope for the
  project
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
* **functional:** enforces functional programming concepts to batch processing
  * **pure-tasks:** akin to pure functions as units of computation with no side-effects
  * **immutability:** of all data blocks if enforced
  * **idempotency:** all tasks use a set of blocks as input, and outputs one block,
    given the same task version and input blocks, will always give the same output block
* **data aware:**
  * **block level statistics:** EToiLe, can compute statistics as it commits
  new blocks of data specified statistics about the data based on a stats
  rule engine. These statistics can be leveraged to enforce data quality
  checks and for anomaly detection
* **time window aware:** heterogenous scheduling batch sizes are
  fully supported, and mixing schedule in a pipeline is allowed. The DAG
  references time-related blocks as specified in task definition. A task
  may specify a dependency on 24 individual hourly blocks as well as it's
  own previous execution's block
* **historical:** In theory, the full target state of the entire data warehouse is
  declared. Provided the raw source blocks, the state of the warehouse at any point can
  be fully recomputed. Note that it is also possible to declare different logic to
  be applied to different, **non-overlapping** time periods on the same dataset.
  Historical jobs not only exist in source control, but are serialized into
  the database for quick retrieval where
  needed (show history in the UI). Optionally all raw source data can
  be marked as such and kept indefinitely, to insure that along with logic
  history being systematically kept, all states in history are
  "auditable" and reproducible
* **garbage collection / retention:** overwritten blocks can be kept around
  and archived for reproducibility. When re-computing occurs, the user can
  specify the retention policy for the overwritten block (delete, keep forever, keep for N days, ...)
* **namespacing / sandboxing:** beyond the main "production" namespace, users
  can spawn subdags into alternative namespaces. Someone may want to execute
  a certain portion in their own developement namespace using alternate logic
  but source data out of production. Subdags expressions can be defined,
  and portions of subdags can be overwritten.
* **logical nodes with bindings:** possibility to attach arbitrary logical nodes
  of different types for say- reports and dashboards
* **stream abstractions:** while streams don't really comply to the
  functional paradigm associated with batch processing, they can be represented
  here logically for lineage and enventualy consumed and landed into batches.
  Many of the guarantees and features of the framework will be limited to
  batch nodes, but the framework can be batch-aware
* **virtual / physical:** each node can be flagged as virtual or physical,
  and potentially decisions around what needs to be materialized can be
  done manually, algorithmically, or a mix of both

# Guarantees & potential
All of this metadata will make for a limitless UI.

* Navigate your current and historical DAGs of computation
* Navigate your table statistics
* Everything is fully reproduceable
* It's just one big DAG, no logical boundaries
* effortless to add columns
* intricate rules around garbage collection 

# Parser / transform /  DSL
* Use [a subset] of SQLAlchemy's SQL expression language?
* Eventually write a SQL parser
* Most common SQL operations (inner join, left join, full outer join, union all, groupbing) would be supported
* Bolt in support for common, translatable DATE functions (probably just pick a decent, known one say MySQL as a base, write translations)
* Add support for hints for things like
  * "DIMENSION JOIN", which implies one-to-many relationship
* special handling around schedule-related special (partitioned) date field

## Semantics & Concepts
* **dataset** is a table or collection of blocks that have a forward compatible schema
* **task** is a piece of logic that source from a collection of blocks and target a single block
* **run** is a single schedule and is time bound, it represents the left bound of the time interval of related task runs
* **task run** is a single run of a single task
* **block commit** is a block that was existed at some point in time, it may be active, archived, or deleted and
  has an effective duration (start and open ended timestamp)
* **subdag** is any subset of the graph
* **column sets** is a set of columns that can be referenced as a group and evolve over time
  adding a new column to a set may be all you have to
* **metric expression** corresponds
* **hoarding-mode** if all raw data (logs, db scrapes, externaly sourced data is
  marked as such, the system can insure that all history on those table is
  hoarded forever and thus all derived datasets in history can be recomputed
  at will (but at a cost!)

## Decisions to be made
* schedule centric?! enforce time-constrained schedules as opposed to batch
  ids (most likely YES)
* use a graph database? GraphQL? (most likely NO)

## Tangents
* Allow for specifying logical, dynamic named subdags. A logical subdag
  may be "all descendants of X", or "lineage between A and B", or
  "all of the ancestors of A and B"
* all dates are truncated on schedule
* targets can be virtual or materialized
* configs HAVE to be serializable so that they can stored in a database.
While the current target for the warehouse lives in source control, the history
of what ran is made available

## Subpartition handling
Somehow subpartioning beyond the schedule-related partition needs to be
possible. Logically though, EToiLe would only be aware of the first-level
partioning.

# A compiler?
Conceptually **EToiLe** is a "data compiler" where:
  * individual data blocks (think of them as partitions) are "compiled" (ie computed)
  * instead of a DAG of files to be compiled, we have a DAG of data blocks
  * when a file is changed, the compiler knows that the downstream blocks need to be recomputed
  * data quality is enforced, akin to unit-tests

# Reltionship with Airflow?
Perhaps Airflow would know how to schedule and run EToiLe specs. EToiLe
should still be self-standing and not require Airflow.

# DB models

#task
task_SHA
target (ix)
task_json
is_current

#task run
target
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
