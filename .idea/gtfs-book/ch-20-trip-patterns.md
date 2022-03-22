## 20. Trip Patterns**

In a GTFS feed, a route typically has multiple trips that start and
finish at the same stops. If you are looking to reduce the size of the
data stored, then converting data from `stop_times.txt` into a series
of reusable patterns is an excellent way to do so.

For two trips to share a common pattern, the following must hold true:

* The stops visited and the order in which they are visited must be the same
* The time differences between each stop must be the same.

The following table shows some fictional trips to demonstrate this.

| **Stop** | **Trip 1** | **Trip 2** | **Trip 3** |
| :------- | :--------- | :--------- | :--------- |
| S1       | 10:00:00   | 10:10:00   | 10:20:00   |
| S2       | 10:02:00   | 10:13:00   | 10:22:00   |
| S3       | 10:05:00   | 10:15:00   | 10:25:00   |
| S4       | 10:06:00   | 10:18:00   | 10:26:00   |
| S5       | 10:10:00   | 10:21:00   | 10:30:00   |

In a GTFS feed, this would correspond to 15 records in
`stop_times.txt`. If you look more closely though, you can see the
trips are very similar. The following table shows the differences
between each stop time, instead of the actual time.

| **Stop** | **Trip 1**      | **Trip 2**      | **Trip 3**      |
| :------- | :-------------- | :-------------- | :-------------- |
| S1       | 00:00:00        | 00:00:00        | 00:00:00        |
| S2       | 00:02:00 (+2m)  | 00:03:00 (+3m)  | 00:02:00 (+2m)  |
| S3       | 00:05:00 (+5m)  | 00:05:00 (+5m)  | 00:05:00 (+5m)  |
| S4       | 00:06:00 (+6m)  | 00:08:00 (+8m)  | 00:06:00 (+6m)  |
| S5       | 00:10:00 (+10m) | 00:11:00 (+11m) | 00:10:00 (+10m) |

You can see from this table that the first and third trip, although they
start at different times, have the same offsets between stops (as well
as stopping at identical stops).

Instead of using a table to store stop times, you can store patterns. By
storing the ID of the pattern with each trip, you can reduce the list of
stop times in this example from 15 to 10. As only time offsets are
stored for each patterns, the trip starting time also needs to be saved
with each trip.

You could use SQL such as the following to model this.

```sql
CREATE TABLE trips (
  trip_id TEXT,
  pattern_id INTEGER,
  start_time TEXT,
  start_time_secs INTEGER
);

CREATE TABLE patterns (
  pattern_id INTEGER,
  stop_id TEXT,
  time_offset INTEGER,
  stop_sequence INTEGER
);
```

The data you would store for trips in this example is shown in the
following table.

| `trip_id` | `pattern_id` | `start_time` | `start_time_secs` |
| :-------- | :----------- | :----------- | :---------------- |
| T1        | 1            | 10:00:00     | 36000             |
| T2        | 2            | 10:10:00     | 36600             |
| T3        | 1            | 10:20:00     | 37200             |

***Note:** The above table includes start_time_secs, which is an integer
value representing the number of seconds since the day started. Using
the hour, minutes and seconds in start_time, this value is `H * 3600 + M * 60 + S`.*

In the `patterns` table, you would store data as in the following
table.

| `pattern_id` | `stop_id` | `time_offset` | `stop_sequence` |
| :----------- | :-------- | :------------ | :-------------- |
| 1            | S1        | 0             | 1               |
| 1            | S2        | 120           | 2               |
| 1            | S3        | 300           | 3               |
| 1            | S4        | 360           | 4               |
| 1            | S5        | 600           | 5               |
| 2            | S1        | 0             | 1               |
| 2            | S2        | 180           | 2               |
| 2            | S4        | 300           | 3               |
| 2            | S5        | 480           | 4               |
| 2            | S6        | 660           | 5               |

As you can see, this represents an easy way to significantly reduce the
amount of data stored. You could have tens or hundreds of trips each
sharing the same pattern. When you scale this to the entire feed, this
could reduce, say, 3 million records to about 200,000.

***Note:** This is a somewhat simplified example, as there is other data
available in `stop_times.txt` (such as separate arrival/departure times,
drop-off type and pick-up type). You should take all of this data into
account when determining how to allocate patterns.*

### Updating Trip Searches

Changing your model to reuse patterns instead of storing every stop time
means your data lookup routines must also be changed.

For example, to find all stop times for a given trip, you must now find
the pattern using the following SQL query.

```sql
SELECT * FROM patterns
  WHERE pattern_id = (SELECT pattern_id FROM trips WHERE trip_id = 'YOUR_TRIP_ID')
  ORDER BY stop_sequence;
```

If you want to determine the arrival/departure time, you must add the
offset stored for the pattern record to the starting time stored with
the trip. This involves joining the tables and adding `time_offset` to
`start_time_secs`, as shown in the following query.

```sql
SELECT t.start_time_secs + p.time_offset, p.stop_id
  FROM patterns p, trips t
  WHERE p.pattern_id = t.pattern_id
  AND t.trip_id = 'YOUR_TRIP_ID'
  ORDER BY p.stop_sequence;
```

### Other Data Reduction Methods

There are other ways you can reduce the amount of data, such as only
using patterns to store the stops (and not timing offsets), and then
storing the timings with each trip record. A technique such as this
further reduces the size of the database, but the trade-off is that
querying the data becomes slightly more complex.

Hopefully you can see that by using the method described in this chapter
there are a number of ways to be creative with GTFS data, and that you
must make decisions when it comes to speed, size, and ease of querying
data.
