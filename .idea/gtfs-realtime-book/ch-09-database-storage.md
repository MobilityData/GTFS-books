## 9. Storing Feed Data in a Database

This chapter will demonstrate how to save data from a GTFS-realtime feed
into an SQLite database. In order to look up trips, stops, routes and
other data from the GTFS-realtime feed, this SQLite database will also
contain the data from the corresponding GTFS feed.

To do this, the `GtfsToSql` and `GtfsRealTimeToSql` tools which have
been written for the purpose of this book and the preceding book, *The
Definitive Guide to GTFS* ([http://gtfsbook.com](http://gtfsbook.com/)),
will be used.

GTFS and GTFS-realtime feeds from the MBTA in Boston
(<https://openmobilitydata.org/p/mbta>) will also be used.

### Storing GTFS Data in an SQLite Database

*The Definitive Guide to GTFS* demonstrated how to populate an SQLite
database using `GtfsToSql`. Here is an abbreviated version of the
steps required to do so.

First, download `GtfsToSql` from
<https://github.com/OpenMobilityData/GtfsToSql>. This is a Java command-line
application to import a GTFS feed into an SQLite database.

The pre-compiled `GtfsToSql` Java archive can be downloaded from its
GitHub repository at
<https://github.com/OpenMobilityData/GtfsToSql/tree/master/dist>.

Next, download the MBTA GTFS feed, available from
<http://www.mbta.com/uploadedfiles/MBTA_GTFS.zip>.

```
$ curl http://www.mbta.com/uploadedfiles/MBTA_GTFS.zip -o gtfs.zip

$ unzip gtfs.zip -d mbta/
```

To create an SQLite database from this feed, the following command can
be used:

```
$ java -jar GtfsToSql.jar -s jdbc:sqlite:./db.sqlite -g ./mbta -o
```

***Note:** The -o flag enables the recording of additional useful data in
the database. For instance, each entry in the trips table will contain
the departure and arrival time (which would otherwise only be available
by looking up the `stop_times` table).*

This may take a minute or two to complete (you will see progress as it
imports the feed and then creates indexes), and at the end you will have
a GTFS database in a file called `db.sqlite`. You can then query this
database with the command-line `sqlite3` tool, as shown in the
following example:

```
$ sqlite3 db.sqlite

sqlite> SELECT agency_id, agency_name, agency_url, agency_lang FROM agency;

2|Massport|http://www.massport.com|EN

1|MBTA|[http://www.mbta.com](http://www.mbta.com/)|EN
```

***Note:** For more information about storing and querying GTFS data,
please refer to *The Definitive Guide to GTFS*, available from
[http://gtfsbook.com](http://gtfsbook.com/).*

### Storing GTFS-realtime Data in an SQLite Database

Once the GTFS data has been imported into the SQLite database, you can
then start importing GTFS-realtime data. For this you can use the
`GtfsRealTimeToSql` tool specifically written to import data into an
SQLite database.

***Note:** Technically you do not need the GTFS data present in the
database as well, but having it makes it far simpler to resolve trip,
route and stop data.*

The GTFS data will change infrequently (perhaps every few weeks or
months), while the realtime data can change several times per minute.
`GtfsRealTimeToSql` will frequently update a feed, according to the
refresh time specified. Each time it updates, the data saved on the
previous iteration is deleted, as that data is no longer the most
up-to-date data.

To get started, download `GtfsRealTimeToSql` from its GitHub
repository at <https://github.com/OpenMobilityData/GtfsRealTimeToSql>. Just
like `GtfsToSql`, this is a Java command-line application. The
pre-compiled `GtfsRealTimeToSql` Java archive can be downloaded from
<https://github.com/OpenMobilityData/GtfsRealTimeToSql/tree/master/dist>.

Since the `db.sqlite` database contains the MBTA GTFS feed, you can
now run `GtfsRealTimeToSql` with one of MBTA's GTFS-realtime feeds.
For instance, their vehicle positions feed is located at
<http://developer.mbta.com/lib/gtrtfs/Vehicles.pb>.

```
java -jar GtfsRealTimeToSql.jar \
  -u "http://developer.mbta.com/lib/gtrtfs/Vehicles.pb" \
  -s jdbc:sqlite:./db.sqlite \
  -refresh 15
```

When you run this command, the vehicle positions feed will be retrieved
every 15 seconds (specified by the `-refresh` parameter), and the data
will be saved into the `db.sqlite` SQLite database.

MBTA also have a trip updates feed and a service alerts feed. In order
to load each of these feeds you will need to run `GtfsRealTimeToSql`
three separate times. To allow it to run in the background, add the
`-d` parameter (to make it run as a server daemon), and background it
using **&**.

To load the vehicle positions in the background, stop the previous
command, then run the following command instead.

```
java -jar GtfsRealTimeToSql.jar \
  -u "http://developer.mbta.com/lib/gtrtfs/Vehicles.pb" \
  -s jdbc:sqlite:./db.sqlite \
  -refresh 15 \
  -d &
```

Now you can also load the trip updates into the same `db.sqlite` file.
MBTA's trip updates feed is located at
<http://developer.mbta.com/lib/gtrtfs/Passages.pb>. The following
command reloads the trip updates every 30 seconds:

```
java -jar GtfsRealTimeToSql.jar \
  -u "http://developer.mbta.com/lib/gtrtfs/Passages.pb" \
  -s jdbc:sqlite:./db.sqlite \
  -refresh 30 \
  -d &
```

***Note:** You may prefer more frequent updates (such as 15 seconds), or
even less frequent (such as 60 seconds). The more frequently you update,
the greater your server utilization will be.*

Finally, to load the service alerts feed, use the same command, but now
with the feed located at
<http://developer.mbta.com/lib/GTRTFS/Alerts/Alerts.pb>. Generally,
service alerts are updated infrequently by providers, so you can use a
refresh time such as five or ten minutes (300 or 600 seconds).

```
java -jar GtfsRealTimeToSql.jar \
  -u "http://developer.mbta.com/lib/gtrtfs/Alerts/Alerts.pb" \
  -s jdbc:sqlite:./db.sqlite \
  -refresh 300 \
  -d &
```

These three feeds will continue to be reloaded until you terminate their
respective processes.

### Querying Vehicle Positions

When using `GtfsRealTimeToSql`, vehicle positions are stored in a
table called `gtfs_rt_vehicles`. If you want to retrieve positions
for, say, the route with an ID of 742, you can run the following query:

```
$ sqlite3 db.sqlite

sqlite> SELECT route_id, trip_id, trip_date, trip_time, trip_sr, latitude, longitude
  FROM gtfs_rt_vehicles
  WHERE route_id = 742;
```

This query returns the trip descriptor and GPS coordinates for all
vehicles on the given route. The following table shows sample results
from this query.

| `route_id` |  `trip_id` |  `trip_date` | `trip_time` | `trip_sr` | `latitude` | `longitude` |
| :--------- | :--------- | :----------- | :---------- | :-------- | :--------- | :---------- |
| 742        |  25900860  |  20150315    |             | 0         | 42.347309  | -71.040359  |
| 742        |            |              |             |           | 42.331505  | -71.065590  |

***Note:** The trip_sr column corresponds to the trip's schedule
relationship value.*

This particular snapshot of vehicle position data shows two different
trips for route 742, which corresponds to the SL2 Silver Line.

```
sqlite> SELECT route_short_name, route_long_name FROM routes WHERE
route_id = 742;

SL2|Silver Line SL2
```

The first five columns retrieved are the trip identifier fields, which
are used to link a vehicle position with a trip from the GTFS feed. The
first row is simple: the `trip_id` value can be used to look up the
row from the `trips` table.

```
sqlite> SELECT service_id, trip_headsign, direction_id, block_id

FROM trips

WHERE trip_id = '25900860';
```

The results from this query are as follows.

| `service_id`                 | `trip_headsign` | `direction_id` | `block_id` |
| :--------------------------- | :-------------- | :------------- | :--------- |
| BUSS12015-hbs15017-Sunday-02 | South Station   | 1              | S742-61    |

***Note:** To see all fields available in this or any other table
(including `gtfs_rt_vehicles`), use the .schema command in SQLite. For
instance, enter the command `.schema gtfs_rt_vehicles`.*

One helpful thing MBTA does is to include the trip starting date when
they include the trip ID. This helps to disambiguate trips that may
start or finish around midnight or later.

The second vehicle position is another matter. This record does not
include any trip descriptor information other than the route ID. This
means you cannot reliably link this vehicle position with a specific
trip from the GTFS feed.

However, knowing the route ID and the vehicle's coordinates may be
enough to present useful information to the user: "A bus on the Silver
Line SL2 route is at this location." You cannot show them if the vehicle
is running late, early or on-time, but you can show the user where the
vehicle is.

### Querying Trip Updates

The `GtfsRealTimeToSql` tool stores trip update data in two tables:
one for the main trip update data (such as the trip descriptor), and
another to store each individual stop time event that belongs within
each update.

For example, to retrieve all trip updates for the route with ID 742 once
again, you could use the following query:

```sql
SELECT route_id, trip_id, trip_date, trip_time, trip_sr, vehicle_id, vehicle_label
  FROM gtfs_rt_trip_updates
  WHERE route_id = '742';
```

This query will return data similar to the following table:

| `route_id` | `trip_id` | `trip_date` | `trip_time` | `trip_sr` | `vehicle_id` | `vehicle_label` |
| :--------- | :-------- | :---------- | :---------- | :-------- | :----------- | :-------------- |
| 742        | 25900860  | 20150315    |             | 0         | y1106        | 1106            |
| 742        | 25900856  | 20150315    |             | 0         |              |                 |
| 742        | 25900858  | 20150315    |             | 0         |              |                 |

Each of these returned records has corresponding records in the
`gtfs_rt_trip_updates_stoptimes` table that includes the
arrival/departure estimates for various stops on the trip.

When using `GtfsRealTimeToSql`, an extra column called `update_id`
is included on both tables so they can be linked together. For example,
to retrieve all updates for the route ID 742, you can join the tables
together as follows:

```sql
SELECT trip_id, arrival_time, arrival_delay, departure_time, departure_delay, stop_id, stop_sequence
  FROM gtfs_rt_trip_updates_stoptimes
  JOIN gtfs_rt_trip_updates USING (update_id)
  WHERE route_id = '742';
```

MBTA's trip updates feed only includes a stop time prediction for the
next stop. This means that each trip in the results only has one
corresponding record. You must then apply the delay to subsequent stop
times for the given trip.

| `trip_id` | `arrival_time` | `arrival_delay` | `departure_time` | `departure_delay` | `stop_id` | `stop_sequence` |
| :-------- | :------------- | :-------------- | :--------------- | :---------------- | :-------- | :-------------- |
| 25900860  |                | 30              |                  |                   | 74617     | 10              |
| 25900856  |                |                 |                  | 0                 | 74611     | 1               |
| 25900858  |                |                 |                  | 0                 | 31255     | 1               |

There are several things to note in these results. Firstly, MBTA do not
provide a timestamp for `arrival_time` and `departure_time`; they
only provide the delay offsets that can be compared to the scheduled
time in the GTFS feed.

Secondly, the second and third trips listed have not yet commenced. You
can deduce this just by looking at this table, since the next stop has
stop sequence of `1` (and therefore there are no stops before it).

The first trip is delayed by 30 seconds. In other words, it will arrive
30 seconds later than it was scheduled. The data received from the
GTFS-realtime feed does not actually indicate the arrival timestamp, but
you can look up the corresponding stop time from the GTFS feed to
determine this.

The following query demonstrates how to look up the arrival time:

```sql
SELECT s.stop_name, st.arrival_time
  FROM stop_times st, trips t, stops s
  WHERE st.trip_index = t.trip_index
  AND st.stop_index = s.stop_index
  AND s.stop_id = '74617'
  AND t.trip_id = '25900860'
  AND st.stop_sequence = 10;
```

***Note:** The `GtfsToSql` tool used to import the GTFS feed adds fields
such as trip_index and stop_index in order to speed up data searching.
There is more discussion on the rationale of this in *The Definitive
Guide to GTFS*, available from
[http://gtfsbook.com](http://gtfsbook.com/).*

This query joins the `stop_times` table to both the `trips` and
`stops` table in order to look up the corresponding arrival time. The
final three rows in the query contain the values returned above (the
`stop_id`, `trip_id` and the `stop_sequence`), to find the
following record:

```
South Station Silver Line - Inbound|21:17:00
```

This means the scheduled arrival time for South Station Silver Line is
9:17:00 PM. Since the estimate indicates a delay of 30 seconds, the new
arrival time is 9:17:30 PM.

***Note:** Conversely, if the delay was -30 instead of 30, the vehicle
would be early and arrive at 9:16:30 PM.*

One thing not touched upon in this example is the stop time's schedule
relationship. The `gtfs_rt_trip_updates_stoptimes` also includes a
column called `rship`, which is used to indicate the schedule
relationship for the given stop. If this value was skipped (a value of
`1`), then it means the vehicle will stop here.

### Determining Predictions Using Blocks

In GTFS, the `block_id` value in **trips.txt** is used to indicate a
series of one or more trips undertaken by a single vehicle. In other
words, once it gets to the end of one trip, it starts a new trip from
that location. If a given trip is very short, a vehicle may perform up
to fifty or one hundred trips in a single day.

This can be useful for determining estimates for future trips that may
not be included in the GTFS feed. For instance, if a trip is running 30
minutes late, then it is highly likely that subsequent trips on that
block will also be running late.

Referring back to the trip data returned in the above example, the
following data can be retrieved from the GTFS feed:

```sql
SELECT trip_id, block_id, service_id, departure_time, arrival_time
  FROM trips
  WHERE trip_id IN ('25900860', '25900856', '25900858')
  ORDER BY departure_time;
```

This returns the following data.

| `trip_id` | `block_id` | `service_id`                 | `departure_time` | `arrival_time` |
| :-------- | :--------- | :--------------------------- | :--------------- | :------------- |
| 25900860  | S742-61    | BUSS12015-hbs15017-Sunday-02 | 21:04:00         | 21:17:00       |
| 25900856  | S742-61    | BUSS12015-hbs15017-Sunday-02 | 21:18:00         | 21:28:00       |
| 25900858  | S742-61    | BUSS12015-hbs15017-Sunday-02 | 21:35:00         | 21:48:00       |

Each of these trips have the same values for `block_id` and
`service_id`, meaning the same vehicle will complete all three trips.
The data indicates that the first trip is running 30 seconds late, so
its arrival time will be 21:17:30.

Because the second trip is scheduled to depart at 21:18:00, there is a
buffer time to catch up (in other words, the first trip is only 30
seconds late, so hopefully it will not impact the second trip).

If, for instance, the second trip ran 10 minutes late (and therefore
arrived at 21:38:00 instead of 21:28:00), you could reasonably assume
the third trip would begin at about 21:38:00 instead of 21:35:00 (about
three minutes late).

### Querying Service Alerts

When using `GtfsRealTimeToSql` to store service alerts, data is stored
in three tables. The main service alert information is stored in
`gtfs_rt_alerts`. Since there can be any number of time ranges or
affected entities for any given alert, there is a table to hold time
ranges (`gtfs_rt_alerts_timeranges`) and one to hold affected entities
(`gtfs_rt_alerts_entities`).

In order to link this data together, each alert has an `alert_id`
value (created by `GtfsRealTimeToSql`) which is also present for any
corresponding time range and affected entities records.

To retrieve service alerts from the database, you can use the following
query:

```sql
SELECT alert_id, header, description, cause, effect FROM gtfs_rt_alerts;
```

At the time of writing, this yields the following results. The
description has been shortened as they are quite long and descriptive in
the original feed.

| `alert_id` | `header`                     | `description`                                              |  `cause` | `effect` |
| :--------- | :--------------------------- | :--------------------------------------------------------- | :------- | :------- |
| 6          | Route 11 detour              | Route 11 outbound detoured due to ...                      |  1       | 4        |
| 11         | Ruggles elevator unavailable | ... Commuter Rail platform to the lobby is unavailable ... |  9       | 7        |
| 33         | Extra Franklin Line service  |                                                            |  1       | 5        |

**Note: **A cause value of 1 corresponds to UNKNOWN_CAUSE, while 9
corresponds to MAINTENANCE. An effect value of 4 corresponds to DETOUR,
7 corresponds to OTHER_EFFECT, while 5 corresponds to
ADDITIONAL_SERVICE.

To determine the timing of these alerts, look up the
`gtfs_rt_alerts_timeranges` for the given alerts.

```
$ export TZ=America/New_York

$ sqlite3 db.sqlite

sqlite> SELECT alert_id, datetime(start, 'unixepoch', 'localtime'), datetime(finish, 'unixepoch', 'localtime')
  FROM gtfs_rt_alerts_timeranges
  WHERE alert_id IN (6, 11, 33);
```

***Note:** In this example, the system timezone has been temporarily
changed to America/New_York (the timezone specified in MBTA's
agency.txt file) so the timestamps are formatted correctly. You may
prefer instead to format the start and finish timestamps using your
programming language of choice.*

| `alert_id` | `start` (formatted) | `finish` (formatted) |
| :--------- | :------------------ | :------------------- |
| 6          | 2015-02-17 15:56:52 |                      |
| 11         | 2015-03-18 06:00:00 | 2015-03-18 16:00:00  |
| 11         | 2015-03-19 06:00:00 | 2015-03-19 16:00:00  |
| 33         | 2015-03-16 10:58:38 | 2015-03-18 02:30:00  |

These dates indicate the following:

* The alert with an ID of 6 began on February 17 and has no specified end date.
* The alert with an ID of 11 will occur over two days between 6 AM and 4 PM.
  *The alert with an ID of 33 will last for almost two days.

Finally, to determine which entities (routes, trips, stops) are affected
by these alerts, query the `gtfs_rt_alerts_entities` table.

```sql
SELECT alert_id, agency_id, route_id, route_type, stop_id, trip_id, trip_start_date, trip_start_time, trip_rship
  FROM gtfs_rt_alerts_entities WHERE alert_id IN (6, 11, 33);
```

This results in the following data, describing only an entity for the
final service alert.

| `alert_id` | `agency_id` | `route_id`  | `route_type` | `stop_id` | `trip_id` | `trip_start_date` | `trip_start_time` | `trip_rship` |
| :--------- | :---------- | :---------- | :----------- | :-------- | :-------- | :---------------- | :---------------- | :----------- |
| 6          | 1           | CR-Franklin | 2            |           |           |                   |                   |              |

Using the `route_id` value, you can find the route from the GTFS feed:

```sql
SELECT agency_id, route_type, route_long_name, route_desc
  FROM routes WHERE route_id = 'CR-Franklin';
```

This query will result in the following data:

| `agency_id` | `route_type` | `route_long_name` | `route_desc`  |
| :---------- | :----------- | :---------------- | :------------ |
| 1           | 2            | Franklin Line     | Commuter Rail |

In effect, this means that if you have a web site or app displaying the
schedule for the Franklin line, this alert should be displayed so the
people who use the line are aware of the change.

***Note:** Even though the `agency_id` and `route_type` values are included,
you do not really need these since the `route_id` can only refer to one
row in the GTFS feed. If instead you wanted to refer to ALL rail lines,
the alert would have the `route_id` value blank but keep the `route_type`
value.*

