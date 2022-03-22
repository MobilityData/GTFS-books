## 4. Routes (routes.txt)

*This file is ***required*** to be included in GTFS feeds.*

A route is a group of trips that are displayed to riders as a single
service.

| Field  | Required? | Description |
| :----- | :-------- | :---------- |
| `route_id` | Required | An ID that uniquely identifies the route. |
| `agency_id` | Optional | The ID of the agency a route belongs to, as it appears in `agency.txt`. Only required if there are multiple agencies in the feed. |
| `route_short_name` | Required | A nickname or code to represent this service. If this is left empty then the `route_long_name` must be included. |
| `route_long_name` | Required | The route full name. If this is left empty then the `route_short_name` must be included. |
| `route_desc` | Optional | A description of the route, such as where and when the route operates. |
| `route_type` | Required | The type of transportation used on a route (such as bus, train or ferry). See below for more information. |
| `route_url` | Optional | A URL of a web page that describes this particular route. |
| `route_color` | Optional | If applicable, a route can have a color assigned to it. This is useful for systems that use colors to identify routes. This value is a six-character hexadecimal number (for example, `FF0000` is red). |
| `route_text_color` | Optional | For routes that specify the `route_color`, a corresponding text color should also be specified. |

### Sample Data

The following extract is taken from the TriMet GTFS feed
(<https://openmobilitydata.org/p/trimet>).

| `route_id` | `route_short_name` | `route_long_name`            | `route_type` |
| :--------- | :----------------- | :--------------------------- | :----------- |
| `1`        | `1`                | `Vermont`                    | `3`          |
| `4`        | `4`                | `Division / Fessenden`       | `3`          |
| `6`        | `6`                | `Martin Luther King Jr Blvd` | `3`          |

This sample shows three different bus routes for the greater Portland
area. The `route_type` value of `3` indicates they are buses. See
the next section for more information about route types in GTFS.

There is no agency ID value in this feed, as TriMet is the only agency
represented in the feed.

The other thing to note about this data is that TriMet use the same
value for both `route_id` and `route_short_name`. This is very
useful, because it means if you have a user that wants to save
information about a particular route you can trust the `route_id`
value. Unfortunately, this is not the case in all GTFS feeds. Sometimes,
the `route_id` value may change with every version of a feed (or at
least, semi-frequently). Additionally, some feeds may also have multiple
routes with the same `route_short_name`. This can present challenges
when trying to save user data.

### Route Types

To indicate a route's mode of transport, the `route_type` column is
used.

| Value | Description       |
| :---- | :---------------- |
| `0`   | Tram / Light Rail |
| `1`   | Subway / Metro    |
| `2`   | Rail              |
| `3`   | Bus               |
| `4`   | Ferry             |
| `5`   | Cable Car         |
| `6`   | Gondola           |  
| `7`   | Funicular         |

Agencies may interpret the meaning of these route types differently. For
instance, some agencies specify their subway service as rail (value of
`2` instead of `1`), while some specify their trains as light rail
(`0` instead of `2`).

These differences between agencies occur mainly because of the vague
descriptions for each of these route types. If you use Google Transit to
find directions, you may notice route types referenced that are
different to those listed above. This is because Google Transit also
supports additional route types. You can read more about these
additional route types at
<https://support.google.com/transitpartners/answer/3520902?hl=en>.

Very few GTFS feeds made available to third-party developers actually
make use of these values, but it is useful to know in case you come
across one that does. For instance, Sydney Buses include their school
buses with a route type of `712`, while other buses in the feed have
route type `700`.

