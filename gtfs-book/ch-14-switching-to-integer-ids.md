## 14. Switching to Integer IDs

There are a number of instances in a GTFS feed where IDs are used, such
as to identify routes, trips, stops and shapes. There are no specific
guidelines in GTFS as to the type of data or length an ID can be. As
such, IDs in some GTFS feeds maybe anywhere up to 30 or 40 characters
long.

Using long strings as IDs is extremely inefficient as they make the size
of a database much larger than it needs to be, as well as making
querying the data much slower.

To demonstrate, consider a GTFS feed where trip IDs are 30 characters
long. If there are 10,000 trips, each with an average of 30 stops in
`stop_times.txt`, then the IDs alone take up 9.3 MB of storage.
Realistically speaking, you need to index the `trip_id` field in order
to look up a trip's stop times quickly, which uses even more space.

The following SQL statements show how you might represent GTFS without
optimizing the identifiers. For brevity, not all fields from the GTFS
feed are included here.

```sql
CREATE TABLE trips (
  trip_id TEXT,
  route_id TEXT,
  service_id TEXT
);

CREATE INDEX trips_trip_id ON trips (trip_id);

CREATE INDEX trips_route_id ON trips (route_id);

CREATE INDEX trips_service_id ON trips (service_id);

CREATE TABLE stop_times (
  trip_id TEXT,
  stop_id TEXT,
  stop_sequence INTEGER
);

CREATE INDEX stop_times_trip_id ON stop_times (trip_id);

CREATE INDEX stop_times_stop_id on stop_times (stop_id);
```

If you were to add an integer column to `trips` called, say,
`trip_index`, then you can reference that value from `stop_times`
instead of `trip_id`. The following SQL statements show this.

```sql
CREATE TABLE trips (
  trip_id TEXT,
  trip_index INTEGER,
  route_id TEXT,
  service_id TEXT
);

CREATE INDEX trips_trip_id ON trips (trip_id);
CREATE INDEX trips_trip_index ON trips (trip_index);
CREATE INDEX trips_route_id ON trips (route_id);
CREATE INDEX trips_service_id ON trips (service_id);

CREATE TABLE stop_times (
  trip_index INTEGER,
  stop_id TEXT,
  stop_sequence INTEGER
);

CREATE INDEX stop_times_trip_index ON stop_times (trip_index);

CREATE INDEX stop_times_stop_id on stop_times (stop_id);
```

This results in a significant space saving (when you consider how large
`stop_times` can be), as well as being far quicker to look up stop
times based on a trip ID. Note that the original `trip_id` value is
retained so it can be referenced if required.

Without adding `trip_index`, you would use the following query to find
stop times given a trip ID.

```sql
SELECT * FROM stop_times
  WHERE trip_id = 'SOME_LONG_TRIP_ID'
  ORDER BY stop_sequence;
```

With the addition of `trip_index`, you need to first find the record
in `trips`. This can be achieved using the following query. This is a
small sacrifice compared to performing string comparison on all stop
times.

```sql
SELECT * FROM stop_times
  WHERE trip_index = (
    SELECT trip_index FROM trips WHERE trip_id = 'SOME_LONG_TRIP_ID'
  )
  ORDER BY stop_sequence;
```

You can make the same change for the other IDs in the feed, such as
`route_id` and `stop_id`. For these columns you still keep (and
index) the original values in `routes` and `stops` respectively,
since you may still need to look up records based on these values.

***Note:** Even though this book recommends optimizing feeds in this
manner, the remainder of examples in this book only use their original
IDs, in order to simplify the examples and to ensure compatibility with
the `GtfsToSql` tool introduced previously.*

