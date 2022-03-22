## 9. Trip Shapes (shapes.txt)

*This file is ***optional*** in a GTFS feed.*

Each trip in `trips.txt` can have a shape associated with it. The
shapes.txt file defines the points that make up an individual shape in
order to plot a trip on a map. Two or more records in `shapes.txt`
with the same `shape_id` value define a shape.

The amount of data stored in this file can be quite large. In
*Optimizing Shapes* there are some strategies to efficiently
reduce the amount of shape data.

| Field  | Required? | Description |
| :----- | :-------- | :---------- |
| `shape_id` | Required | An ID to uniquely identify a shape. Every point for a shape contains the same value. |
| `shape_pt_lat` | Required | The latitude for a given point in the range of `-90` to `90`. |
| `shape_pt_lon` | Required | The longitude for a given point in the range of `-180` to `180`. |
| `shape_pt_sequence` | Required | A non-negative number that defines the ordering for points in a shape. A value must not be repeated within a single shape. |
| `shape_dist_traveled` | Optional | This value represents how far along a shape a particular point exists. This is a distance in a unit such as feet or kilometers. This unit must be the same as that used in `stop_times.txt`. |

### Sample Data

The following table shows a portion of a shape from the TriMet GTFS
feed. It is a portion of the shape that corresponds to the sample data
in the `stop_times.txt` section.

| `shape_id` | `shape_pt_lat` | `shape_pt_lon` | `shape_pt_sequence` | `shape_dist_traveled` |
| :--------- | :------------- | :------------- | :------------------ | :-------------------- |
| 185328     | 45.52291       | -122.677372    | 1                   | 0.0                   |
| 185328     | 45.522921      | -122.67737     | 2                   | 3.7                   |
| 185328     | 45.522991      | -122.677432    | 3                   | 34.0                  |
| 185328     | 45.522992      | -122.677246    | 4                   | 81.5                  |
| 185328     | 45.523002      | -122.676567    | 5                   | 255.7                 |
| 185328     | 45.523004      | -122.676486    | 6                   | 276.4                 |
| 185328     | 45.523007      | -122.676386    | 7                   | 302.0                 |
| 185328     | 45.523024      | -122.675386    | 8                   | 558.4                 |
| 185328     | 45.522962      | -122.67538     | 9                   | 581.0                 |

In this sample data, the `shape_dist_traveled` is listed in feet.
There is no way to specify in a GTFS feed which units are used for this
column -- it could be feet, miles, meters, kilometers. In actual fact,
it does not really matter, just as long as the units are the same as in
`stop_times.txt`.

If you need to present a distance to your users (such as how far you
need to travel on a bus), you can calculate it instead by adding up the
distance between each point and formatting it based on the user's
locale settings.

### Point Sequences

In most GTFS feeds the `shape_pt_sequence` value starts at 1 and
increments by 1 for every subsequent point. Additionally, points are
typically listed in order of their sequence.

You should not rely on these two statements though, as this is not a
requirement of GTFS. Many transit agencies have automated systems that
export their GTFS from a separate system, which can sometimes result in
an unpredictable output format.

For instance, a trip that has stop times listed with the sequences `1`, `2`,
`9`, `18`, `7`, `3` is perfectly valid.

### Distance Travelled

The `shape_dist_traveled` column is used so you can programmatically
determine how much of a shape to draw when showing a map to users of
your web site or app. If you use techniques in *Optimizing Shapes*
to reduce the file size of shape data, then it becomes difficult to
use this value.

Alternatively, you can calculate portions of shapes by determining which
point in a shape travels closest to the start and finish points of a
trip.

