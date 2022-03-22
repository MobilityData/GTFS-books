## 5. Trips (trips.txt)

*This file is ***required*** to be included in GTFS feeds.*

The `trips.txt` file contains trips for each route. The specific stop
times are specified in `stop_times.txt`, and the days each trip runs
on are specified in `calendar.txt` and `calendar_dates.txt`.

| Field  | Required? | Description |
| :----- | :-------- | :---------- |
| `route_id` | Required | The ID of the route a trip belongs to as it appears in `routes.txt`. |
| `service_id` | Required | The ID of the service as it appears in `calendar.txt` or `calendar_dates.txt`, which identifies the dates on which a trip runs. |
| `trip_id` | Required | A unique identifier for a trip in this file. This value is referenced in `stop_times.txt` when specifying individual stop times. |
| `trip_headsign` | Optional | The text that appears to passengers as the destination or description of the trip. Mid-trip changes to the headsign can be specified in `stop_times.txt`. |
| `trip_short_name` | Optional | A short name or code to identify the particular trip, different to the route's short name. This may identify a particular train number or a route variation. |
| `direction_id` | Optional | Indicates the direction of travel for a trip, such as to differentiate between an inbound and an outbound trip. |
| `block_id` | Optional | A block is a series of trips conducted by the same vehicle. This ID is used to group 2 or more trips together. |
| `shape_id` | Optional | This value references a value from `shapes.txt` to define a shape for a trip. |
| `wheelchair_accessible` | Optional | `0` or blank indicates unknown, while `1` indicates the vehicle can accommodate at least one wheelchair passenger. A value of `2` indicates no wheelchairs can be accommodated. |

### Sample Data

Consider the following extract, taken from the `trips.txt` file of the
TriMet GTFS feed (<https://openmobilitydata.org/p/trimet>).

| `route_id` | `service_id` | `trip_id`   | `direction_id` | `block_id` | `shape_id` |
| ---------- | ------------ | ----------- | -------------- | ---------- | ---------- |
| 1          | W.378        | 4282257     | 0              | 103        | 185327     |
| 1          | W.378        | 4282256     | 0              | 101        | 185327     |
| 1          | W.378        | 4282255     | 0              | 102        | 185327     |
| 1          | W.378        | 4282254     | 0              | 103        | 185327     |

This data describes four individual trips for the "Vermont" bus route
(this was determined by looking up the `route_id` value in
`routes.txt`). While the values for `trip_id`, `block_id` and
`shape_id` are all integers in this particular instance, this is not a
requirement. Just like `service_id`, there may be non-numeric
characters.

As each of these trips is for the same route and runs in the same
direction (based on `direction_id`) they can all be represented by the
same shape. Note however that this is not always be the case as some
agencies may start or finish trips for a single route at different
locations depending on the time of the day. If this were the case, then
the trip's shape would differ slightly (and therefore have a different
shape to represent it in `shapes.txt`).

Although this example does not include `trip_headsign`, many feeds do
include this value. This is useful for indicating to a passenger where
the trip is headed. When the trip headsign is not provided in the feed,
you can determine the destination by using the final stop in a trip.

**Tip:** If you are determining the destination based on the final stop,
you can either present the stop name to the user, or you can
reverse-geocode the stop's coordinates to determine the its locality.

### Blocks

In the preceding example, each trip has a value for `block_id`. The
first and the last trips here both have a `block_id` value of `103`.
This indicates that the same physical vehicle completes both of these
trips. As each of these trips go in the same direction, it is likely
that they start at the same location.

This means there is probably another trip in the feed for the same block
that exists between the trips listed here. It would likely travel from
the finishing point of the first trip (`4282257`) to the starting
point of the other trip (`4282254`). If you dig a little deeper in the
feed you will find the trip shown in the following table.

| `route_id` | `service_id` | `trip_id` | `direction_id` | `block_id` | `shape_id` |
| :--------- | :----------- | :-------- | :------------- | :--------- | :--------- |
| 1          | W.378        | 4282270   | 1              | 103        | 185330     |

This is a trip traveling in the opposite direction for the same block.
It has a different shape ID because it is traveling in the opposite
direction; a shape's points must advance in the same direction a trip's
stop times do.

***Note:** You should perform some validation when grouping trips
together using block IDs. For instance, if trips share a block_id value
then they should also have the same service_id value. You should also
check that the times do not overlap; otherwise the same vehicle would be
unable to service both trips.*

If you dig even further in this feed, there are actually seven different
trips all using block `103` for the **W.378** service period. This
roughly represents a full day's work for a single vehicle.

For more discussion on blocks and how to utilize them effectively, refer
to *Working With Trip Blocks*.

### Wheelchair Accessibility

Similar to `stops.txt`, you can specify the wheelchair accessibility
of a specific trip using the `wheelchair_accessible` field. While many
feeds do not provide this information (often because vehicles in a fleet
can be changed at the last minute, so agencies do not want to guarantee
this information), your wheelchair-bound users will love you if you can
provide this information.

As mentioned in the section on `stops.txt`, it is equally important to
tell a user that a specific vehicle cannot accommodate wheelchairs as to
when it can. Additionally, if the stops in a feed also have wheelchair
information, then both the stop and trip must be wheelchair accessible
for a passenger to be able to access a trip at the given stop.

### Trip Direction

One of the optional fields in `trips.txt` is `direction_id`, which
is used to indicate the general direction a vehicle is traveling. At
present the only possible values are `0` to represent "inbound" and
`1` to represent "outbound". There are no specific guidelines as to
what each value means, but the intention is that an inbound trip on a
particular route should be traveling in the opposite direction to an
outbound trip.

Many GTFS feeds do not provide this information. In fact, there are a
handful of feeds that include two entries in `routes.txt` for each
route (one for each direction).

One of the drawbacks of `direction_id` is that there are many routes
for which "inbound" or "outbound" do not actually mean anything. Many
cities have loop services that start and finish each trip at the same
location. Some cities have one or more loops that travel in both
directions (generally indicated by "clockwise loop" and
"counter-clockwise loop", or words to that effect). In these instances,
the `direction_id` can be used to determine which direction the route
is traveling.

### Trip Short Name

The `trip_short_name` field that appears in `trips.txt` is used to
provide a vehicle-specific code to a particular trip on a route. Based
on GTFS feeds that are currently in publication, it appears there are
two primary use-cases for this field:

* Specifying a particular train number for all trains on a route
* Specifying a route "sub-code" for a route variant.

### Specifying a Train Number

For certain commuter rail systems, such as SEPTA in Philadelphia or MBTA
in Boston, each train has a specific number associated with it. This
number is particularly meaningful to passengers as trains on the same
route may have different stopping patterns or even different features
(for instance, only a certain train may have air-conditioning or
wheelchair access).

Consider the following extract from SEPTA's rail feed
(<https://openmobilitydata.org/p/septa>).

| `route_id` | `service_id` | `trip_id`    | `trip_headsign`          | `block_id` | `trip_short_name` |
| ---------- | ------------ | ------------ | ------------------------ | ---------- | ----------------- |
| AIR        | S5           | AIR_1404_V25 | Center City Philadelphia | 1404       | 1404              |
| AIR        | S1           | AIR_402_V5   | Center City Philadelphia | 402        | 402               |
| AIR        | S1           | AIR_404_V5   | Center City Philadelphia | 404        | 404               |
| AIR        | S5           | AIR_406_V25  | Center City Philadelphia | 406        | 406               |

In this data, there are four different trains all heading to the same
destination. The `trip_short_name` is a value that can safely be
presented to users as it has meaning to them. In this case, you could
present the first trip to passengers as:

**"Train 1404 on the Airport line heading to Center City
Philadelphia."**

In this particular feed, SEPTA use the same value for
`trip_short_name` and for `block_id`, because the train number
belongs to a specific train. This means after it completes the trip to
Center City Philadelphia it continues on. In this particular feed, the
following trip also exists:

**"Train 1404 on the Warminster line heading to Glenside."**

You can therefore think of the `trip_short_name` value as a
"user-facing" version of `block_id`.

### Specifying a Route Sub-Code

The other use-case for `trip_short_name` is for specifying a route
sub-code. For instance, consider an agency that has a route with short
name `100` that travels from stop `S1` to stop `S2`. At night the
agency only has a limited number of routes running, so they extend this
route to also visit stop `S3` (so it travels from `S1` to `S2`
then to `S3`). As it is a minor variation of the main path, the agency
calls this trip `100A`.

The agency could either create a completely separate entry in
`routes.txt` (so they would have `100` and `100A`), or they can
override the handful of trips in the evening by setting the
`trip_short_name` to `100A`. The following table shows how this
example might be represented.

| `route_id` | `trip_id` | `service_id` | `trip_short_name` |
| ---------- | --------- | ------------ | ----------------- |
| 100        | T1        | C1           |                   |
| 100        | T2        | C1           |                   |
| 100        | T3        | C1           |                   |
| 100        | T4        | C1           | 100A              |

In this example the `trip_short_name` does not need to be set for the
first three trips as they use the `route_short_name` value from
`routes.txt`.

### Specifying Bicycle Permissions

A common field that appears in many GTFS fields is
`trip_bikes_allowed`, which is used to indicate whether or not
passengers are allowed to take bicycles on board. This is useful for
automated trip planning when bicycle options can be included in the
results.

The way this field works is similar to the wheelchair information; `0`
or empty means no information provided; `1` means no bikes allowed;
while `2` means at least one bike can be accommodated.

**Note:** Unfortunately, this value is backwards when you compare it to
wheelchair accessibility fields. For more discussion on this matter,
refer to the topic on the Google Group for GTFS Changes
(<https://groups.google.com/d/topic/gtfs-changes/rEiSeKNc4cs/discussion>).

