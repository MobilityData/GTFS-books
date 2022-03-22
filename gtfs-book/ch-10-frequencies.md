## 10. Repeating Trips (frequencies.txt)

*This file is ***optional*** in a GTFS feed.*

In some cases a route may repeat a particular stopping pattern every few
minutes (picture a subway line that runs every 5 minutes). Rather than
including entries in `trips.txt` and `stop_times.txt` for every
single occurrence, you can include the trip once then define rules for
it to repeat for a period of time.

Having a trip repeat only works in the case where the timing between
stops remains consistent for all stops. Using `frequencies.txt`, you
use the relative times between stops alongside a calculated starting
time for the trip in order to determine the specific stop times.

| Field  | Required? | Description |
| :----- | :-------- | :---------- |
| `trip_id` | Required | The ID of the trip as it appears in `trips.txt` that is being repeated. A single trip can appear multiple times for different time ranges. |
| `start_time` | Required | The time at which a given trip starts repeating, in `HH:MM:SS` format. |
| `end_time` | Required | The time at which a given trip stops repeating, in `HH:MM:SS` format. |
| `headway_secs` | Required | The time in seconds between departures from a given stop during the time range. |
| `exact_times` | Optional | Whether or not repeating trips should be exactly scheduled. See below for discussion. |

### Sample Data

The following sample data is taken from Société de transport de Montréal
(STM) in Montreal
(<https://openmobilitydata.org/p/societe-de-transport-de-montreal>).

| `trip_id`                | `start_time` | `end_time` | `headway_secs` |
| :----------------------- | :----------- | :--------- | :------------- |
| 13S_13S_F1_1_2_0.26528   | 05:30:00     | 07:25:30   | 630            |
| 13S_13S_F1_1_6_0.34167   | 07:25:30     | 08:40:10   | 560            |
| 13S_13S_F1_1_10_0.42500  | 08:40:10     | 12:19:00   | 505            |
| 13S_13S_F1_1_7_0.58750   | 12:19:00     | 15:00:00   | 460            |
| 13S_13S_F1_1_11_0.66875  | 15:00:00     | 18:23:00   | 420            |
| 13S_13S_F1_1_5_0.78889   | 18:23:00     | 21:36:35   | 505            |

Each of the trips listed here have corresponding entries in
`trips.txt` and `stop_times.txt` (more on that shortly). This data
can be interpreted as follows.

* The first trip runs every 10m 30s from 5:30am until 7:25am.
* The second trip runs every 9m 20s from 7:25am until 8:40am, and so on.

The following table shows some of the stop times for the first trip
(`departure_time` is omitted here for brevity, since it is identical
to `arrival_time`).

| `trip_id`               | `stop_id` | `arrival_time` | `stop_sequence` |
| :---------------------- | :-------- | :------------- | :-------------- |
| 13S_13S_F1_1_2_0.26528 | 18        | 06:22:00       | 1               |
| 13S_13S_F1_1_2_0.26528 | 19        | 06:22:59       | 2               |
| 13S_13S_F1_1_2_0.26528 | 20        | 06:24:00       | 3               |
| 13S_13S_F1_1_2_0.26528 | 21        | 06:26:00       | 4               |

As this trip runs to the specified frequency, the specific times do not
matter. Instead, the differences are used. For the above stop times,
there is a 59 second gap between the first and second time, a 61 second
gap between the second and third, and a 120 second gap between the third
and fourth.

The stop times for the first frequency record (10.5 minutes apart) can
be calculated as follows.

* `05:30:00`, `05:30:59`, `05:32:00`, `05:34:00`
* `05:40:30`, `05:41:29`, `05:42:30`, `05:44:30`
* `05:51:00`, `05:51:59`, `05:53:00`, `05:55:00`
* ...
* `07:25:30`, `07:26:29`, `07:27:30`, `07:29:30`

### Specifying Exact Times

In the file definition at the beginning of this chapter there is an
optional field called `exact_times`. It may not be immediately clear
what this field means, so to explain it better, consider the frequency
definitions in the following table.

| `trip_id` | `start_time` | `end_time` | `headway_secs` | `exact_times` |
| :-------- | :----------- | :--------- | :------------- | :------------ |
| T1        | 09:00:00     | 10:00:00   | 300            | 0             |
| T2        | 09:00:00     | 10:00:00   | 300            | 1             |

These two frequencies are the same, with only the `exact_times` value
different. The first (`T1`) should be presented in a manner such as:

**"Between 9 AM and 10 AM this trip departs every 5 minutes."**

The second trip (`T2`) should be presented as follows:

**"This trip departs at 9 AM, 9:05 AM, 9:10 AM, ..."**

While ultimately the meaning is the same, this difference is used in
order to allow agencies to represent their schedules more accurately.
Often, schedules that convey to passengers that they will not have to
wait more than five minutes do so without having to explicitly list
every departure time.

