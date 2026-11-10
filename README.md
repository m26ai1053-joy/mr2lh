# From MapReduce to Lakehouse

Material from two talks, given in this order:

1. **Spark as a query engine**: how Spark runs a query, from SQL text to tasks on executors.
2. **Delta Lake**: how a folder of Parquet files becomes a table you can update, version and trust.

The second talk picks up where the first ends (the last Spark slide asks *"When does a directory of files stop being enough?"*), so go through them in order.

```
spark-query-engine/
  From MapReduce to Lakehouse.pdf     slides, talk 1
  spark-query-walkthrough.html        interactive walkthrough of query planning
lakehouse-delta/
  delta_lake_talk.pdf                 slides, talk 2
  01_files_to_table.ipynb             notebooks, run alongside the slides
  02_versions_and_rules.ipynb
  02a_two_writers.ipynb
  03_skipping_and_merge.ipynb
  04_change_feed.ipynb
  delta-merge-two-phases.html         interactive visualization of MERGE
```

---

## Part 1 · Spark query engine

No setup needed. Both files open in a browser or PDF viewer.

| Step | Open | What it covers |
|---|---|---|
| 1 | `From MapReduce to Lakehouse.pdf`, slides 1–17 | The evolving-dataset problem, storage and engine choices, Spark basics (RDD, DataFrame, partitions and cores), file formats, `map` vs `mapPartitions`, join strategies, handling skew |
| 2 | `spark-query-walkthrough.html` (at slide 18, *Spark Query Planning & Optimizer*) | Step through one query: parsing and analysis, logical optimization (pushdown), choosing a physical join, whole-stage codegen, Adaptive Query Execution and skew splitting. Use the controls on each page to change the data assumptions and watch the plan change |
| 3 | Same PDF, slides 19–22 | Accelerators (Photon, Comet, Gluten), approximations and sketches, and the closing question that leads into Delta Lake |

---

## Part 2 · Delta Lake

The slides are split into four modules. Each notebook belongs to a point in the slides: read up to that point, run the notebook, then carry on. Every notebook ends with a question that the next slide or notebook answers.

### Running the notebooks

The notebooks were written for **Google Colab** and need nothing installed beforehand.

1. Open a notebook in Colab: *File → Open notebook → GitHub*, paste this repo's URL, or upload the `.ipynb` file.
2. Run the first cell, **Setup — run before the session**. It installs `pyspark==3.5.3` and `delta-spark==3.3.2`, installs Java if missing, and starts a local Spark session. Allow a minute or two.
3. Run the remaining cells top to bottom.

Each notebook is self-contained: setup wipes and recreates its working folder (`/tmp/delta_talk`), so you can run any notebook on its own or re-run one from the start.

To run locally instead, use Python 3.9+ with Java 17 on your `PATH` (the setup cell's `apt-get` fallback only works on Debian/Ubuntu as root), then open the notebooks in Jupyter or PyCharm.

### Order

| Step | Open | What it covers |
|---|---|---|
| 1 | `delta_lake_talk.pdf`, slides 1–7 (**Module 1**) | Why a folder of Parquet files isn't a table; Delta = Parquet + a transaction log |
| 2 | `01_files_to_table.ipynb` | Create a Delta table and look at what lands on disk. Update one row and see it become *remove + add* in a single commit. Show that a stray file in the folder is ignored, delete a row, read the table's history |
| 3 | Slides 8–11 (**Module 2**) | Versions and MVCC: never overwrite, always add a version. Optimistic concurrency: two writers, one wins; readers never wait |
| 4 | `02a_two_writers.ipynb` | Race two jobs against each other: two inserts, then two updates to *different* users. See which one commits, why the other is rejected, and how it retries |
| 5 | Slide 12 | A partner sends a wrong age |
| 6 | `02_versions_and_rules.ipynb` | Find the `age = 999` row with time travel, undo it with `RESTORE` (and check that no data was copied), then add a `CHECK` constraint so it can't happen again |
| 7 | Slide 13 | ACID: what the log and versions give us |
| 8 | Slides 14–16 (**Module 3**) | Data skipping with min/max stats, and why file layout matters |
| 9 | `03_skipping_and_merge.ipynb`, Part 1 | 16,000 users in two layouts. Read the per-file stats from the log and count how many files Spark actually opens for `id = 750` and for a filter on `interest` |
| 10 | Slides 17–18 | MERGE: a batch with one update and one new user |
| 11 | `delta-merge-two-phases.html` | Interactive view of MERGE's two phases: skip with stats, find the files that actually match, then rewrite only those |
| 12 | `03_skipping_and_merge.ipynb`, Part 2 | Do the same MERGE by hand (skip, inner join, full outer join), then run the real `MERGE` and check which files it rewrote |
| 13 | Slides 19–20 (**Module 4**) | Recap of skipping and clustering; three kinds of read: current, time travel, change feed |
| 14 | `04_change_feed.ipynb` | Turn on Change Data Feed, make three changes, read them back as `update_preimage` / `update_postimage` / `delete` / `insert` rows, and update a dashboard incrementally |
| 15 | Slides 21–22 | Retention limits on the change feed, and a recap of every problem from slide 4 and what fixed it |
