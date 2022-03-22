## 8. Fare Definitions (fare_attributes.txt & fare_rules.txt)

*These files are ***optional*** in a GTFS feed, but any rules specified
must reference a fare attributes record.*

These two files define the types of fares that exist in a system,
including their price and transfer information. The attributes of a
particular fare exist in `fare_attributes.txt`, which has the
following columns.

| Field  | Required? | Description |
| :----- | :-------- | :---------- |
| `fare_id` | Required | A value that uniquely identifies a fare listed in this file. It must only appear once in this file. |
| `price` | Required | This field specifies the cost of the fare. For instance, if a trip costs $2 USD, then this value should be either `2.00` or `2`, and the `currency_type` should be `USD`. |
| `currency_type` | Required | This is the currency code that `price` is specified in, such as USD for United States Dollar. |
| `payment_method` | Required | This indicates when the fare is to be paid. `0` means it can be paid on board, while `1` means it must be paid before boarding. |
| `transfers` | Required | The number of transfers that may occur. This must either be empty (unlimited) transfers, or the number of transfers. |
| `transfer_duration` | Optional | This is the number of seconds a transfer is valid for. |

The following table shows the specification for `fare_rules.txt`,
which defines the rules used to apply a fare to a particular trip.

| Field  | Required? | Description |
| :----- | :-------- | :---------- |
| `fare_id` | Required | This is the ID of the fare that a rule applies to as it appears in `fare_attributes.txt`. |
| `route_id` | Optional | The ID of a route as it appears in `routes.txt` for which this rule applies. If there are several routes with the same fare attributes, there may be a row in `fare_rules.txt` for each route. |
| `origin_id` | Optional | This value corresponds to a `zone_id` value from `stops.txt`. If specified, this means a trip must begin in the given zone in order to qualify for this fare. |
| `destination_id` | Optional | This value corresponds to a `zone_id` value from `stops.txt`. If specified, this means a trip must begin in the given zone in order to qualify for this fare. |
| `contains_id` | Optional | This value corresponds to a `zone_id` value from `stops.txt`. If specified, this means a trip must pass through every `contains_id` zone for the given fare (in other words, several rules may need to be checked). |

Note that aside from `fare_id`, all fields are optional in this file.
This means some very complex rules can be made (especially when
transfers come into the calculation). The following URL has discussion
about different rules and some complex fare examples:

<https://code.google.com/p/googletransitdatafeed/wiki/FareExamples>

Refer to *Calculating Fares* for discussion about the
algorithm for calculating fares for trips both with and without
transfers.

### Sample Data

The following data is taken from the TriMet GTFS feed
(<https://openmobilitydata.org/p/trimet>). Firstly, the data from the
`fare_attributes.txt` file.

| `fare_id` | `price` | `currency` | `payment_method` | `transfers` | `transfer_duration` |
| :-------- | :------ | :--------- | :--------------- | :---------- | :-------------------|
| B         | 2.5     | USD        | 0                |             | 7200                |
| R         | 2.5     | USD        | 1                |             | 7200                |
| BR        | 2.5     | USD        | 0                |             | 7200                |
| RB        | 2.5     | USD        | 1                |             | 7200                |
| SC        | 1       | USD        | 1                | 0           |                     |
| AT        | 4       | USD        | 1                | 0           |                     |
| VT        | 0       | USD        | 1                | 0           |                     |

The data from the `fare_rules.txt` file is shown in the following
table.

| `fare_id` | `route_id` | `origin_id` | `destination_id` | `contains_id` |
| :-------- | :--------- | :---------- | :--------------- | :------------ |
| B         |            | B           |                  | B             |
| R         |            | R           |                  | R             |
| BR        |            | B           |                  | B             |
| BR        |            | B           |                  | R             |
| RB        |            | R           |                  | B             |
| RB        |            | R           |                  | R             |
| SC        | 193        |             |                  |               |     
| SC        | 194        |             |                  |               |     
| AT        | 208        |             |                  |               |     
| VT        | 250        |             |                  |               |     

In this sample data, TriMet have named some of their fares the same as
the zones specified in `stops.txt`. In this particular feed, bus stops
have a `zone_id` of `B`, while rail stops have `R`.

The fares in this file are as follows:

* Fare `B`. If you start at a bus stop (`zone_id` value of `B`),
  you can buy your ticket on board (`payment_method` of `0`). You
  may transfer an unlimited number (empty transfers value) for 2 hours
  (`transfer_duration` of `7200`). The cost is $2.50 USD.
* Fare `R`. If you start at a rail stop, you must pre-purchase your
  ticket. You may transfer an unlimited number of times to other rail
  services for up to 2 hours. The cost is $2.50 USD.

The `BR` fare describes a trip that begins on a bus then transfers to
a rail service (while `RB` is the opposite). This fare is not be
matched if the passenger does not travel on both, as all `contains_id`
values must be matched in order to apply a fare.

The other fares (`SC`, `AT` and `VT`) all apply to their
respective `route_id` values, regardless of start and finish stops.
Tickets must be pre-purchased, and transfers are not allowed. The `VT`
fare (which corresponds to TriMet's Vintage Trolley) is free to ride
since it has a price of `0`.

### Assigning Fares to Agencies

One of the extensions available to `fare_attributes.txt` is to include
an `agency_id` column. This is to limit a specific fare to only routes
from the specified agency, in the case where a feed has multiple
agencies.

This is useful because there may be two agencies in a feed that define
fares with no specific rules (in other words, the fare applies to all
trips). If the price differs, then GTFS dictates that the cheapest fare
is always applied. Using `agency_id` means these fares can be
differentiated accordingly.

For more information about this extension, refer to the Google Transit
GTFS Extensions page at
<https://support.google.com/transitpartners/answer/2450962>.

