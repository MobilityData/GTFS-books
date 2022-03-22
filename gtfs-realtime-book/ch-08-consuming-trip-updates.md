## 8. Consuming Trip Updates

Of the three message types in GTFS-realtime, trip updates are the most
complex. A single trip update can contain a large quantity of data and
is used to transform the underlying schedule. Trips can be modified in a
number of ways: trips can be canceled, stops can be skipped, and arrival
times can be updated.

This chapter will show you how to consume trip updates and will discuss
real-world scenarios and how they can be represented using trip updates.

Similar to service alerts and vehicle positions, you can loop over the
`FeedEntity` objects from `getEntityList()` to access and then
process `TripUpdate` objects.

```java
for (FeedEntity entity : fm.getEntityList()) {
  if (entity.hasTripUpdate()) {
    TripUpdate tripUpdate = entity.getTripUpdate();

    processTripUpdate(tripUpdate);
  }
}
```

### Timestamp

Just like with vehicle position updates, trip updates include a
timestamp value. This indicates when the real-time data was updated,
including any subsequent arrival/departure estimates contained within
the trip update.

You can read this value into a native `java.util.Date` object as
follows:

```java
if (tripUpdate.hasTimestamp()) {
  Date timestamp = new Date(tripUpdate.getTimestamp() * 1000);

}
```

**Note:** The value is multiplied by 1,000 because the java.util.Date
class accepts milliseconds, whereas GTFS-realtime uses whole seconds.

Two of the ways you can use the timestamp value are:

* When telling your users arrival estimates, you can also show the
  timestamp so they know when the estimate was made. A newer estimate
  (e.g. one minute old) is likely to be more reliable than an older
  one (e.g. ten minutes old).
* You can also use the timestamp to decide whether or not to keep the
  estimate. For instance, if for some reason a feed did not refresh in
  a timely manner and all of the estimates were hours old, you could
  simply skip over them as they would no longer provide meaningful
  information.

### Trip Information

Each trip update contains necessary data so the included estimates can
be linked to a trip in the corresponding GTFS feed.

There are three ways estimate data can be provided in a trip update:

* The trip update contains estimates for all stops.
* The trip update contains estimates for some stops.
* The trip has been canceled, so there is no stop information.

If the trip has been added (that is, it is not a part of the schedule in
the GTFS feed), then the trip descriptor has no use in resolving the
trip back to the GTFS feed.

However, if a trip update only contains updates for some of the stops,
you must be able to find the entire trip in the GTFS feed so you can
propagate the arrival delay to subsequent stops.

The following code shows how to extract values such as the GTFS trip ID
and route ID from the `TripDescriptor` object.

```java
if (tripUpdate.hasTrip()) {
  TripDescriptor trip = tripUpdate.getTrip();

  if (trip.hasTripId()) {
    String tripId = trip.getTripId();

    // ...
  }

  if (trip.hasRouteId()) {
    String routeId = trip.getRouteId();

    // ...
  }

  if (trip.hasStartDate()) {
    String startDate = trip.getStartDate();

    // ...
  }

  if (trip.hasStartTime()) {
    String startTime = trip.getStartTime();

    // ...
  }

  if (trip.hasScheduleRelationship()) {
    ScheduleRelationship sr = trip.getScheduleRelationship();

    // ...
  }
}
```

### Trip Delay

Although only an experimental part of the GTFS-realtime specification at
the time of writing, a trip update can also contain a delay value. A
positive number indicates the number of seconds the vehicle is late,
while a negative value indicates the number of seconds early. A value of
`0` means the vehicle is on-time.

This value can be used for any stop along the trip that does not
otherwise have an associated `StopTimeEvent` element.

```java
if (tripUpdate.hasDelay()) {
  int delay = tripUpdate.getDelay();

  if (delay == 0) {
    // on time

  }
  else if (delay < 0) {
    // early

  }
  else if (delay > 0) {
    // late

  }
}
```

### Vehicle Identifiers

The `VehicleDescriptor` object contained in a trip update provides a
number of values by which to identify a vehicle. You can access an
internal identifier (not for public display), a label (such as a vehicle
number painted on to a vehicle), or a license plate.

The following code shows how to access these values:

```java
if (tripUpdate.hasVehicle()) {
  VehicleDescriptor vehicle = tripUpdate.getVehicle();

  if (vehicle.hasId()) {
    String id = vehicle.getId();

  }

  if (vehicle.hasLabel()) {
    String label = vehicle.getLabel();

  }

  if (vehicle.hasLicensePlate()) {
    String licensePlate = vehicle.getLicensePlate();

  }
}
```

The vehicle descriptor and the values contained within are all optional.
In the case where this information is not available, you can use the
trip descriptor information provided with each vehicle position to match
up vehicle positions across multiple updates.

### Stop Time Updates

Each trip update contains a number of stop time updates, each of which
is a `StopTimeUpdate` object. You can access each of the
`StopTimeUpdate` objects by calling **getStopTimeUpdateList()**.

```java
for (StopTimeUpdate stopTimeUpdate : tripUpdate.getStopTimeUpdateList()) {
  // ...
}
```

Alternatively, you can loop over each `StopTimeUpdate` object as
follows:

```java
for (int i = 0; i < tripUpdate.getStopTimeUpdateCount(); i++) {
  StopTimeUpdate stopTimeUpdate = tripUpdate.getStopTimeUpdate(i);

  // ...
}
```

Each `StopTimeUpdate` object contains a schedule relationship value
(using the `ScheduleRelationship` class, which is different to that
contained in `TripDescriptor` objects).

The schedule relationship dictates how to use the rest of the data in
the stop time update, as well as which data will be present. If the
schedule relationship value is not present, the value is assumed to be
`SCHEDULED`.

```java
ScheduleRelationship sr;

if (stopTimeUpdate.hasScheduleRelationship()) {
  sr = stopTimeUpdate.getScheduleRelationship();

}
else {
  sr = ScheduleRelationship.SCHEDULED;

}

if (sr.getNumber() == ScheduleRelationship.SCHEDULED_VALUE) {
  // An arrival and/or departure estimate is provided

}
else if (sr.getNumber() == ScheduleRelationship.NO_DATA_VALUE) {
  // No real-time data available in this update

}
else if (sr.getNumber() == ScheduleRelationship.SKIPPED_VALUE) {
  // The vehicle will not stop at this stop

}
```

### Stop Information

In order to determine which stop a stop time update corresponds to,
either the stop ID or stop sequence (or both) must be specified. You can
then look up the stop based on its entry in `stops.txt`, or determine
the stop based on the corresponding entry in `stop_times.txt`.

```java
if (stopTimeUpdate.hasStopId()) {
  String stopId = stopTimeUpdate.getStopId();

}

if (stopTimeUpdate.hasStopSequence()) {
  int sequence = stopTimeUpdate.getStopSequence();

}
```

### Arrival/Departure Estimates

If the `ScheduleRelationship` value is `SKIPPED` there must either
be an arrival or departure object specified. Both the arrival and
departure use the `StopTimeEvent` class, which can be accessed as
follows.

The following code shows how to access these values:

```java
if (sr.getNumber() == ScheduleRelationship.SCHEDULED_VALUE) {
  if (stopTimeUpdate.hasArrival()) {
    StopTimeEvent arrival = stopTimeUpdate.getArrival();

    // Process the arrival
  }

  if (stopTimeUpdate.hasDeparture()) {
    StopTimeEvent departure = stopTimeUpdate.getDeparture();

    // Process the departure
  }
}
```

Each `StopTimeEvent` object contains either an absolute timestamp for
the arrival or departure, or it contains a delay value. The delay value
is relative to the scheduled time in the corresponding GTFS feed.

```java
if (stopTimeUpdate.hasArrival()) {
  StopTimeEvent arrival = stopTimeUpdate.getArrival();

  if (arrival.hasDelay()) {
    int delay = arrival.getDelay();

    // ...
  }

  if (arrival.hasTime()) {
    Date time = new Date(arrival.getTime() * 1000);

    // ...
  }

  if (arrival.hasUncertainty()) {
    int uncertainty = arrival.getUncertainty();

    // ...
  }
}
```

### Trip Update Scenarios

This chapter has shown you how to handle trip update information as it
appears in a GTFS-realtime feed. It is also important to understand the
intent of the data provider when reading these updates. Consider the
following scenarios that can occur frequently in large cities:

1.  **A train needs to skip one or more stops.** Perhaps the station has
    been closed temporarily due to unforeseen circumstances.
2.  **A train that was scheduled to bypass a station will now stop at
    it.** Perhaps there is a temporary delay on the line so the train
    will wait at a stop while the track is cleared.
3.  **A bus is rerouted down a different street.** Perhaps there was a
    car accident earlier and police have redirected traffic.
4.  **A train will stop at a different platform.** For example, instead
    of a train stopping at platform 5 at a given station, it will now
    stop at platform 6. This happens frequently on large train networks
    such as in Sydney.
5.  **A bus trip is completely canceled.** Perhaps the bus has broken
    down and there is no replacement vehicle.
6.  **An unplanned trip is added.** Perhaps there is an unexpectedly
    large number of passengers so extra buses are brought in to clear
    the backlog.

While each provider may represent these scenarios differently in
GTFS-realtime, it is likely each of them would be represented as
follows.

1.  **Modify the existing trip.** Include a `StopTimeUpdate` for each
    stop that is to be canceled with a `ScheduleRelationship` value of
    `SKIPPED`.
2.  **Cancel the trip and add a new one.** Unfortunately, there is no
    way in GTFS-realtime to insert a stop into an existing trip. The
    `ScheduleRelationship` value for the trip would be set to
    `CANCELED` (in the `TripDescriptor`, not in the
    `StopTimeUpdate` elements).
3.  If a bus is rerouted down a different street, how it is handled
    depends on which stops are missed:

    * If no stops are to be made on the new street, cancel the stops
      impacted by the detour, similar to scenario 1.
    * If the bus will stop on the new street to drop passengers off or
      pick them up, then cancel the trip and add a new one, similar to
      scenario 2.

4.  **Cancel the trip and add a new one.** Since it is not possible to
    insert a stop using GTFS-realtime, the existing trip must be
    canceled and a new trip added to replace it if you want the platform
    to be reflected accurately. Note, however, that many data providers
    do not differentiate between platforms in their feeds, so they only
    refer to the parent station instead. In this instance a platform
    change should be communicated using service alerts if it could
    otherwise cause confusion.
5.  **Cancel the trip.** In this instance you would not need to include
    any `StopTimeUpdate` elements for the trip; rather, you would
    specify `CANCELED` in the `ScheduleRelationship` field of
    `TripDescriptor`.
6.  **Add a new trip.** When adding a new trip, all of the stops in the
    trip should also be included (not just the next one). The
    `ScheduleRelationship` for the trip would be set to `ADDED`.

The important takeaway from this section is that if you are telling your
users that a trip has been canceled, you need to make it clear to them
if a new trip has replaced it, otherwise they may not correctly
understand the intent of the data.

In these instances, hopefully the data provider also provides a
corresponding service alert so the reason for the change can be
communicated to passengers.

