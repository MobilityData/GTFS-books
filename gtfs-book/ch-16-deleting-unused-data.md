## 16. Deleting Unused Data**

Once you have imported a GTFS feed into a database, it is possible for
there to be a lot of redundant data. This can ultimately slow down any
querying of that data as well as bloating the size of the database. If
you are making an app where the GTFS database is queried on the device
then disk space and computational time are at a premium, so you must do
what you can to reduce resource usage.

The first thing to check for is expired services. You can do this by
searching `calendar.txt` for entries that expire before today's date.
Be aware though, you also need to ensure there are no
`calendar_dates.txt` entries overriding these services (a service
could have an `end_date` of, say, `20140110`, but also have a
`calendar_dates.txt` entry for `20140131`).

Firstly, find service IDs in `calendar_dates.txt` that are still
active using the following query.

```sql
SELECT service_id FROM calendar_dates WHERE date >= '20140110';
```

***Note:** In order to improve performance of searching for dates, you
should import the date field in `calendar_dates.txt` as an integer, as
well as `start_date` and `end_date` in `calendar.txt`.*

Any services matched in this query should not be removed. You can then
find service IDs in `calendar.txt` with the following SQL query.

```sql
SELECT * FROM calendar WHERE end_date < '20140110'
  AND service_id NOT IN (
    SELECT service_id FROM calendar_dates WHERE date >= '20140110'
  );
```

Before deleting these services, corresponding trips and stop times must
be removed since you need to know the service ID in order to delete a
trip. Likewise, stop times must be deleted before trips since you need
to know the trip IDs to be removed.

```sql
DELETE FROM stop_times WHERE trip_id IN (
   SELECT trip_id FROM trips WHERE service_id IN (
     SELECT service_id FROM calendar WHERE end_date < '20140110'
       AND service_id NOT IN (
         SELECT service_id FROM calendar_dates WHERE date >= '20140110'
       )
    )
  );
```

Now there may be a series of trips with no stop times. Rather than
repeating the above sub-queries, a more thorough way of removing trips
is to remove trips with no stop times.

```sql
DELETE FROM trips WHERE trip_id NOT IN (
  SELECT DISTINCT trip_id FROM stop_times
);
```

With all `service_id` references removed, you can remove the expired
rows from `calendar.txt` using the following SQL query.

```sql
DELETE FROM calendar WHERE end_date < '20140110'
  AND service_id NOT IN (
    SELECT DISTINCT service_id FROM calendar_dates WHERE date >= '20140110'
  );
```

The expired rows in `calendar_dates.txt` can also be removed, which
can be achieved using the following query.

```sql
DELETE FROM calendar_dates WHERE date < '20140110';
```

There may now be some stops that are not used by any trips. These can be
removed using the following query.

```sql
DELETE FROM stops WHERE stop_id NOT IN (
  SELECT DISTINCT stop_id FROM stop_times
);
```

Additionally, you can remove unused shapes and routes using the
following queries.

```sql
DELETE FROM shapes WHERE shape_id NOT IN (
  SELECT DISTINCT shape_id FROM trips
);

DELETE FROM routes WHERE route_id NOT IN (
  SELECT DISTINCT route_id FROM trips
);
```

There are other potential rows that can be removed (such as records in
`transfers.txt` that reference non-existent stops), but hopefully you
get the idea from the previous queries.

