## 19. Calculating Fares**

In order to calculate a fare for a trip in GTFS, you must use data from
`fare_attributes.txt` and `fare_rules.txt`. It is relatively
straightforward to calculate the cost of a single trip (that is,
boarding a vehicle, traveling for a number of stops, then disembarking),
but it becomes much more complicated when you need to take into account
multiple trip segments (that is, one or more transfers).

***Note:** As it stands, many feed providers do not include fare
information. This is because many systems have a unique set of rules
that cannot be modelled with the current structure of fares in GTFS.
Additionally, it is not possible to determine different classes of
pricing (such as having a separate price for adults and children). For
the purposes of this chapter, these limitations are ignored.*

For more discussion on how fares work in GTFS, refer to *Fare
Definitions (`fare_attributes.txt` & `fare_rules.txt`)*.

This chapter first shows you how to calculate fares for a single trip,
then how to account for transfers and for multiple trips.

### Calculating a Single Trip Fare

Much of the logic used when calculating fares requires knowledge of the
zones used in a trip.

***Note:** A zone is a physical area within a transit system that
contains a series of stops. They are used to group trip pricing into key
areas in a system. Some transit systems do not work like this (for
instance, they may measure the physical distance travelled rather than
specific stops), which is one reason why the GTFS fares model does not
work in all cases.*

A zone is defined in GTFS by the `zone_id` column in `stops.txt`. A
single stop can only belong to one zone.

Fares are matched to a trip using a combination of any of the following:

* The route of the trip
* The zone of the stop the passenger boards from
* The zone of the stop the passenger disembarks
* The zone(s) of any stops on the trip that are passed while the
  passenger is on board.

Consider the following simplified data set that may appear in
`stops.txt` and `stop_times.txt`. Assume for this example that the
trip `T1` belongs to a route with an ID of `R1`.

```
stop_id,zone_id
S1,Z1
S2,Z1
S3,Z2
S4,Z3

trip_id,stop_id,stop_sequence
T1,S1,1
T1,S2,2
T1,S3,3
T1,S4,4
```

If a passenger travels from stop S1 to stop S4, then their starting zone
is Z1, their finishing zone is Z3, and the zones they pass through are
Z1, Z2 and Z3.

***Note:** When calculating fares, the start and finish zones are also
included in the zones passed through, so in this example you Z3 is also
considered as a zone that the trip passes through.*

Using this data, you can now calculate matching fares. To do so, you
need to find all fares that match either of the following:

* Fares that have no associated rules.
* Fares that have rules that match the specified trip. If a fare
  defines multiple zones that must be passed through (using
  `contains_id`), then all zones must be matched.

If multiple fares qualify for a trip, then the cheapest fare is the one
to use.

### Finding Fares With No Rules

This is the simplest use-case for fares. You can find all matching fares
with the following SQL.

```sql
SELECT * FROM fare_attributes WHERE fare_id NOT IN (
  SELECT DISTINCT fare_id FROM fare_rules
);
```

If a feed only has `fare_attributes.txt` records with no rules, then
the difference between the fares is in the transfer rules. This section
only covers calculating fares for a single trip with no transfers, so
for now you can just select the cheapest fare using the following SQL.

```sql
SELECT * FROM fare_attributes WHERE fare_id NOT IN (
    SELECT DISTINCT fare_id FROM fare_rules
  )
  ORDER BY price + 0 LIMIT 1;
```

***Note:** You still need to check for fares with one or more rules in
order to find the cheapest price. Also, `0` is added in this query in
order to cast a string to a number. When you roll your own importer you
should instead import this as a numerical value.*

### Finding Fares With Matched Rules

Next you must check against specific rules for a fare. In order to do
this, you need the starting zone, finishing zone, and all zones passed
through (including the start and finish zones).

Referring back to the previous example, if a trip starts at `Z1`,
passes through `Z2` and finishes at `Z3`, you can find fare
candidates (that is, trips that *may* match), using the following SQL
query.

```sql
SELECT * FROM fare_attributes WHERE fare_id IN (
    SELECT fare_id FROM fare_rules
      WHERE (LENGTH(route_id) = 0 OR route_id = 'R1')
      AND (LENGTH(origin_id) = 0 OR origin_id = 'Z1')
      AND (LENGTH(destination_id) = 0 OR destination_id = 'Z3')
      AND (LENGTH(contains_id) = 0 OR contains_id IN ('Z1', 'Z2', 'Z3')
    )
  );
```

This returns a list of fares that may qualify for the given trip. As
some fares have multiple rules, all must be checked. The algorithm to
represent this is as follows.

```
fares = [ result from above query ]

qualifyingFares = [ ]

for (fare in fares) {
  if (qualifies(fare))
    qualifyingFares.add(fare)
}

allFares = qualifyFares + faresWithNoRules

passengerFare = cheapest(allFares)
```

As shown on the final two lines, once you have the list of qualifying
fares, you can combine these with fares that have no rules (from the
previous section) and then determine the cheapest fare.

First though, you must determine if a fare with rules qualifies for the
given trip. If a fare specifies zones that must be passed through, then
all rules must be matched.

***Note:** If a particular rule specifies a different route, start, or
finish than the one you are checking, you do not need to ensure the
`contains_id` matches, since this rule no longer applies. You still need
to check the other rules for this fare.*

The algorithm needs to build up a list of zone IDs from the fare rules
in order to check against the trip. Once this has been done, you need to
check that every zone ID collected from the rules is contained in the
trip's list of zones.

```
qualifies(fare, routeId, originId, destinationId, containsIds) {

  fareContains = [ ]
  
  for (rule in fare.rules) {
    if (rule.contains.length == 0)
      continue
      
    if (rule.route.length > 0 AND rule.route != routeId)
      continue
      
    if (rule.origin.length > 0 AND rule.origin != originId)
      continue
      
    if (rule.desination.length > 0 AND rule.destination != destinationId)
      continue
      
    fareContains.add(rule.containsId);
 }
 
 if (fareContains.size == 0)
   return YES
   
 if (containIds HAS EVERY ELEMENT IN fareContains)
   return YES
 else 
   return NO
}
```

This algorithm achieves the following:

* Only rules that have a value for `contains_id` are relevant. Rules
  that do not have this value fall through and should be considered as
  qualified.
* If the route is specified but not equal to the one being checked, it
  is safe to ignore the rule's `contains_id`. If the route is empty
  or equal, the loop iteration can continue.
* Check for the `origin_id` and `destination_id` in the same
  manner as `route_id`.
* If the route, origin and destination all qualify then store the
  `contains_id` so it can be checked after the loop.

The algorithm returns *yes* if the fare qualifies, meaning you can save
it as a qualifying fare. You can then return the cheapest qualifying
fare to the user.

### Calculating Trips With Transfers

Once you introduce transfers, fare calculation becomes more complicated.
A "trip with a transfer" is considered to be a trip where the passenger
boards a vehicle, disembarks, and then gets on another vehicle. For
example:

* Travel on trip T1 from Stop S1 to Stop S2
* Walk from Stop S2 to Stop S3
* Travel on trip T2 from Stop S3 to Stop S4.

In order to calculate the total fare for a trip with transfers, the
following algorithm is used:

1. Retrieve list of qualifying fares for each trip individually
2. Create a list of every fare combination possible
3. Loop over all combinations and find the total cost
4. Return the lowest cost from Step 3.

Step 1 was covered in *Calculating a Single Trip Fare*, but
you must skip the final step of finding the cheapest fare. This is
because the cheapest fare may change depending on subsequent transfers.
Instead, this step is performed once the cheapest *combination* is
determined.

To demonstrate Step 2, consider the following example:

* The trip on T1 from S1 to S2 yields the following qualifying fares:
  F1, F2.
* The subsequent trip on T2 from S3 to S4 yields the following
  qualifying fares: F3, F4.

Generating every combination of these fares yields the following
possibilities:

* F1 + F3
* F1 + F4
* F2 + F3
* F2 + F4.

Step 3 can now be performed, which involves finding the total cost for
each combinations. As you need to take into account the possibility of
timed transfers (according to the data stored in
`fare_attributes.txt`), you also need to know about the times of these
trips.

The following algorithm can be used to calculate the total cost using
transfer rules. In this example, you would call this function once for
each fare combination.

```
function totalCost(fares) {
  total = 0
 
  for (fare in fares) {
    freeTransfer = NO
 
    if (previousFare ALLOWS TRANSFERS) {
      if (HAS ENOUGH TRANSFERS REMAINING) {
        if (TRANSFER NOT EXPIRED) {
          freeTransfer = YES
        }
      }
    }
 
    if (!freeTransfer)
      total = total + fare.price;
     
    previousFare = fare;
  }
 
  return total;
}
```

Once all combinations have called the `totalCost` algorithm, you will
have a price for each trip. You can then return the lowest price as the
final price for the trip.

