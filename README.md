# StayNest Delta Lakehouse - Session 7

My submission for the Codebasics Session 7 assignment. I did everything in a Databricks Free Edition notebook using PySpark and Spark SQL.

The notebook is `StayNest_S07_assignment_notebook.ipynb`, and I kept all the outputs so you can see the results.

## Setup
I uploaded the CSVs (bookings, hotels, customers, bookings_updates) to a Volume at `/Volumes/workspace/default/staynest` and ran everything on serverless compute. Section 0 prints 12000 / 200 / 200 as expected.

## What I did

**Task 1 - Broadcast join:** I ran `.explain()` on a normal join and then on the same join with `broadcast()`. The second plan shows `BroadcastHashJoin`. Hotels is tiny (200 rows), so Spark just sends a copy to every executor and the big bookings table doesn't need to be shuffled.

**Task 2 - Delta table + history:** I saved bookings as the Delta table `bookings_delta`, then ran an UPDATE (pending to completed) and a DELETE (cancelled rows). `DESCRIBE HISTORY` shows them as versions 0, 1 and 2.

**Task 3 - Time travel and RESTORE:** Version 0 has 12,000 rows, and the current table had fewer after the delete. After `RESTORE ... TO VERSION AS OF 0` the count was back to 12,000. What I learned: RESTORE doesn't delete anything, it shows up as a new version (v3), so the older versions are still there.

**Task 4 - OPTIMIZE and ZORDER:** I ran OPTIMIZE to merge small files, then ZORDER BY city. City works well because it has lots of different values and people filter on it. Status only has 3 values, so ZORDER wouldn't skip any files.

**Tasks 5-7 - Bronze, silver, gold:**
- `bronze_bookings`: raw copy of all 12,000 rows plus an `ingested_at` column
- `silver_bookings`: only completed bookings, joined with the hotel name, category and star rating (I dropped the hotels `city` column so there's only one)
- `gold_city_revenue`: number of bookings and total revenue per city, highest revenue first

**Task 8 - MERGE:** One MERGE statement updates the matching booking_ids and inserts the new ones. The count went from 12,000 to 12,050, which is the 50 new bookings.

## Things I'd say in a review
- RESTORE is safe because it just adds a new commit.
- I'd use MERGE for daily change feeds so only the changed rows get touched.
