## 3. Stops & Stations (stops.txt)

*This file is ***required*** to be included in GTFS feeds.*

The individual locations where vehicles pick up or drop off passengers
are represented by `stops.txt`. Records in this file are referenced in
`stop_times.txt`. A record in this file can be either a stop or a
station. A station has one or more child stops, as indicated using the
`parent_station` value. Entries that are marked as stations may not
appear in `stop_times.txt`.

| Field  | Required? | Description |
| :----- | :-------- | :---------- |
| `stop_id` | Required | An ID to uniquely identify a stop or station. |
| `stop_code` | Optional | A number or short string used to identify a stop to passengers. This is typically displayed at the physical stop or on printed schedules. |
| `stop_name` | Required | The name of the stop as passengers know it by. |
| `stop_desc` | Optional | A description of the stop. If provided, this should provide additional information to the `stop_name` value. |
| `stop_lat` | Required | The latitude of the stop (a number in the range of `-90` to `90`). |
| `stop_lon` | Required | The longitude of the stop (a number in the range of `-180` to `180`). |
| `zone_id` | Optional | This is an identifier used to calculate fares. A single zone ID may appear in multiple stops, but is ignored if the stop is marked as a station. |
| `stop_url` | Optional | A URL that provides information about this stop. It should be specific to this stop and not simply link to the agency's web site. |
| `location_type` | Optional | Indicates if a record is a stop or station. `0` or blank means a stop, `1` means a station. |
| `parent_station` | Optional | If a record is marked as a stop and has a parent station, this contains the ID of the parent (the parent must have a `location_type` of `1`). |
| `stop_timezone` | Optional | If a stop is located in a different time zone to the one specified in `agency.txt`, then it can be overridden here. |
| `wheelchair_boarding` | Optional | A value of `1` indicates it is possible for passengers in wheelchairs to board or alight. A value of `2` means the stop is not wheelchair accessible, while `0` or an empty value means no information is available. If the stop has a parent station, then 0 or an empty value means to inherit from its parent. |

### Sample Data

The following extract is taken from the TriMet GTFS feed
(<https://openmobilitydata.org/p/trimet>).

| `stop_id` |  `stop_code` |  `stop_name`        | `stop_lat`  | `stop_lon`    | `stop_url`                                          |
| :-------- | :----------- | :------------------ | :---------- | :------------ | :---------------------------------------------------|
| `2`       | `2`          | `A Ave & Chandler`  | `45.420595` | `-122.675676` |` <http://trimet.org/arrivals/tracker?locationID=2>` |
| `3`       | `3`          | `A Ave & Second St` | `45.419386` | `-122.665341` |` <http://trimet.org/arrivals/tracker?locationID=3>` |
| `4`       | `4`          | `A Ave & 10th St`   | `45.420703` | `-122.675152` |` <http://trimet.org/arrivals/tracker?locationID=4>` |
| `6`       | `6`          | `A Ave & 8th St`    | `45.420217` | `-122.67307`  |` <http://trimet.org/arrivals/tracker?locationID=6>` |

The following diagram shows how these points look if you plot them onto
a map.

![Stops](images/stops-sample.png)

In this extract, TriMet use the same value for stop IDs and stop codes.
This is useful, because it means the stop IDs are stable (that is, they
do not change between feed versions). This means that if you want to
save a particular stop (for instance, if a user wants to save a
"favorite stop") you can trust that saving the ID will get the job done.

**Note:** This is not always the case though, which means you may have
to save additional information if you want to save a stop. For instance,
you may need to save the coordinates or the list of routes a stop serves
so you can find it again if the stop ID has changed in a future version
of the feed.

### Stops & Stations

Specifying an entry in this file as a *station* is typically used when
there are many stops located within a single physical entity, such as a
train station or bus depot. While many feeds do not offer this
information, some large train stations may have up to 20 or 30
platforms.

Knowing the platform for a specific trip is extremely useful, but if a
passenger wants to select a starting point for their trip, showing them
a list of platforms may be confusing.

Passenger:

**"I want to travel from *Central Station* to *Airport Station*."**

Web site / App:

**"Board at *Central Station platform 5*, disembark at *Airport Station platform 1*."**

In this example, the passenger selects the parent station, but they are
presented with the specific stop so they know exactly where within the
station they need to embark or disembark.

### Wheelchair Accessibility

If you are showing wheelchair accessibility information, it is important
to differentiate between "no access" and "no information", as knowing a
stop is not accessible is as important as knowing it is.

If a stop is marked as being wheelchair accessible, you must check that
trips that visit the stop are also accessible (using the
`wheelchair_accessible` field in `trips.txt`). If the value in
`trips.txt` is blank, `0` or `1` then it is safe to assume the
trip can be accessed. If the stop is accessible and the trip is not,
then passengers in wheelchairs cannot use the trip.

### Stop Features

One of the proposed changes to GTFS is the addition of a file called
`stop_features.txt`. This is used to define characteristics about
stops. The great thing about this file is that it allows you to indicate
to users when a stop has a ticket machine, bike storage, lighting, or an
electronic display with real-time information.

TriMet is one of the few agencies including this file. The following is
a sample of this file.

| `stop_id` | `feature_type` |
| :-------- | :------------- |
| `61`      | `4110`         |
| `61`      | `2310`         |
| `61`      | `5200`         |

This data indicate that stop `61` (NE Alberta & 24th) has a *Printed
Schedule Display* (`4110`), a *Bike Rack* (`2310`) and a *Street
Light* (`5200`).

For more information about this proposal and a list of values and their
meanings, refer to
<https://sites.google.com/site/gtfschanges/proposals/stop-amenity>.

