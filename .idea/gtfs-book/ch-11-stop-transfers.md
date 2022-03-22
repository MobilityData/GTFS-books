## 11. Stop Transfers (transfers.txt)

*This file is ***optional*** in a GTFS feed.*

To define how passengers can transfer between routes at specific stops
feed providers can include `transfers.txt`. This does not mean
passengers cannot transfer elsewhere, but it does indicate if a transfer
is not possible between certain stops, or a minimum time required if
transfer is possible.

| Field  | Required? | Description |
| :----- | :-------- | :---------- |
| `from_stop_id` | Required | The ID of stop as it appears in `stops.txt` where the connection begins. If this references a station, then this rule applies to all stops within the station. |
| `to_stop_id` | Required | The ID of the stop as it appears in `stops.txt` where the connection between trips ends. If this references a station, then this rule applies to all stops within the station. |
| `transfer_type` | Required | `0` or blank means the recommended transfer point, `1` means the secondary vehicle will wait for the first, `2` means a minimum amount of time is required, `3` means transfer is not possible. |
| `min_transfer_time` | Optional | If the `transfer_type` value is `2` then this value must be specified. It indicates the number of seconds required to transfer between the given stops. |

It is also possible that records in this file are specified for
ticketing reasons. For instance, some train stations are set up so that
passengers can transfer between routes without needing to validate their
ticket again or buy a transfer. Other stations that are shared between
those same routes might not have this open transfer area, thereby
requiring you to exit one route fully before buying another ticket to
access the second.

### Sample Data

The following table shows some sample transfer rules from TriMet in
Portland's GTFS feed (<https://openmobilitydata.org/p/trimet>).

| `from_stop_id` | `to_stop_id` | `transfer_type` | `min_transfer_time` |
| :------------- | :----------- | :-------------- | :------------------ |
| 7807           | 5020         | 0               |                     |
| 7807           | 7634         | 0               |                     |
| 7807           | 7640         | 0               |                     |

These rules indicate that if you are transferring from a route that
visits stop `7807` to any route that visits the other stops (`5020`,
`7634` or `7640`), then this is the ideal place to do it.

In other words, if there are other locations along the first route where
you could transfer to the second route, then those stops should not be
used. These rules say this is the best place to transfer.

Consider the transfer rule in the following table, taken from the New
York City Subway GTFS feed (<https://openmobilitydata.org/p/mta/79>).

| `from_stop_id` | `to_stop_id` | `transfer_type` | `min_transfer_time` |
| :------------- | :----------- | :-------------- | :------------------ |
| 121            | 121          | 2               | 180                 |

In this data, the MTA specifies how long it takes to transfer to
different platforms within the same station. The stop with ID 121 refers
to the 86th St station (as specified in `stops.txt`). It has a
`location_type` of `1` and two stops within it (`121N` and
`121S`). The above transfer rule says that if you need to transfer
from `121N` to `121S` (or vice-versa) then a minimum time of 3
minutes (180 seconds) must be allocated.

If you were to calculate the time taken to transfer using the
coordinates of each of these platforms, it would only take a few seconds
as they are physically close to each other. In reality though, you must
exit one platform then walk around and enter the other platform (often
having to use stairs).

