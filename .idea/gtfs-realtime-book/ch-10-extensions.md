## 10. GTFS-realtime Extensions

*Introduction to gtfs-realtime.proto* showed you an extract
from the `gtfs-realtime.proto` file that is used as an input to the
Protocol Buffers `protoc` command.

One of the lines in this extract included the following `extensions`
directive:

```
extensions 1000 to 1999;
```

Each element in a Protocol Buffers entity must be assigned a value. This
is the value used to represent the element in the binary protocol buffer
stream (for instance, the `trip` element was assigned a value of
`1`). The `extensions` directive reserves the values between 1000
and 1999 for external use.

Having these values reserved means that transit agencies are free to
include additional information in their own GTFS-realtime feeds. In this
instance, the agency provides its own `proto` file to use with
`protoc` that builds on the original `gtfs-realtime.proto` file.

At the time of writing, the following extensions are defined:

| Extension ID | Developer |
| :----------- | :-------- |
| 1000         | OneBusAway |
| 1001         | New York City MTA |
| 1002         | Google |
| 1003         | OVapi |
| 1004         | Metra |

You can find this list at
<https://developers.google.com/transit/gtfs-realtime/changes>). As more
agencies release GTFS-realtime feeds this list will grow, since many
agencies have specific requirements in the way their internal systems
work, as well as to facilitate providing data in a way that is
understood by users of the given transport system.

To demonstrate how extensions are specified and used, the following case
study will show you the extension for the New York City subway system.

### Case Study: New York City Subway

One of the agencies that provides an extension is New York City MTA.
They add a number of custom fields to their subway GTFS-realtime feeds.

The following is a slimmed-down version of their Protocol Buffers
definition file, available from
<http://datamine.mta.info/sites/all/files/pdfs/nyct-subway.proto.txt>.

```
option java_package = "com.google.transit.realtime";

import "gtfs-realtime.proto";

message TripReplacementPeriod {
  optional string route_id = 1;
  optional transit_realtime.TimeRange replacement_period = 2;
}

message NyctFeedHeader {
  required string nyct_subway_version = 1;
  repeated TripReplacementPeriod trip_replacement_period = 2;
}

extend transit_realtime.FeedHeader {
  optional NyctFeedHeader nyct_feed_header = 1001;
}

message NyctTripDescriptor {
  optional string train_id = 1;
  optional bool is_assigned = 2;

  enum Direction {
    NORTH = 1;
    EAST = 2;
    SOUTH = 3;
    WEST = 4;
  }

  optional Direction direction = 3;
}

extend transit_realtime.TripDescriptor {
  optional NyctTripDescriptor nyct_trip_descriptor = 1001;
}

// NYCT Subway extensions for the stop time update

message NyctStopTimeUpdate {
  optional string scheduled_track = 1;
  optional string actual_track = 2;
}

extend transit_realtime.TripUpdate.StopTimeUpdate {
  optional NyctStopTimeUpdate nyct_stop_time_update = 1001;
}
```

This file begins by importing the original `gtfs-realtime.proto` file
that was examined in *Introduction to gtfs-realtime.proto*.

It then defines a number of new element types (they choose to prefix
them using `Nyct`, although the only important thing here is that they
don't conflict with the names from `gtfs-realtime.proto`).

This definition file then extends the `TripDescriptor` and
`StopTimeUpdate` element types. Values
`1000` to `1999` are reserved for custom extensions. This is the
reason why the added `nyct_trip_descriptor` and
`nyct_stop_time_update` fields use numbers in this range.

Compiling an Extended Protocol Buffer

In order to make use of these extended fields, you first need to
download and compile the `nyct-subway.proto` file into the same
directory as `gtfs-realtime.proto` (*Compiling gtfs-realtime.proto*).

```
$ cd /path/to/protobuf

$ curl \
  http://datamine.mta.info/sites/all/files/pdfs/nyct-subway.proto.txt \
  -o nyct-subway.proto
```

You can now build a Java class similar to before, but using the
`nyct-subway.proto` file as the input instead of
`gtfs-realtime.proto`:

```
$ protoc \
  --proto_path=/path/to/protobuf \
  --java_out=/path/to/gtfsrt/src \
  /path/to/protobuf/nyct-subway.proto
```

If this command executes successfully, you will now have a file called
`NyctSubway.java` in the **./src/com/google/transit/realtime**
directory (in addition to `GtfsRealtime.java`, which is still
required).

The next section will show you how to use this class.

### Registering Extensions

The process to load an extended GTFS-realtime is the same as a regular
feed (in that you call **parseFrom()** to build the `FeedMessage`
object), but first you must register the extensions.

In Java, this is achieved using the `ExtensionRegistry` class and the
**registerAllExtensions()** helper method.

```
import com.google.protobuf.ExtensionRegistry;

...

ExtensionRegistry registry = ExtensionRegistry.newInstance();

NyctSubway.registerAllExtensions(registry);
```

The `ExtensionRegistry` object is then passed to **parseFrom()** as
the second argument. The following code shows how to open and read the
main New York City subway feed.

***Note:** A developer API key is required to access the MTA
GTFS-realtime feeds. You can register for a key at
[http://datamine.mta.info](http://datamine.mta.info/), then substitute
it into the key variable.*

```java
public class YourClass {
  public void loadFeed() throws IOException {

    String key = "*YOUR_KEY*";

    URL url = new URL("http://datamine.mta.info/mta_esi.php?feed_id=1&key=" + key);

    InputStream is = url.openStream();

    ExtensionRegistry registry = ExtensionRegistry.newInstance();

    NyctSubway.registerAllExtensions(registry);

    FeedMessage fm = FeedMessage.parseFrom(is, registry);

    is.close();

    // ...

  }
}
```

Once the feed has been parsed, you will no longer need to pass the
extension registry around.

### Accessing Extended Protocol Buffer Elements

Earlier it was explained that the general paradigm with reading
GTFS-realtime data is to check the existence of a field using
`hasFieldName()`, and to retrieve the value using
`getFieldName()`.

This also applies to retrieving extended elements, but instead of there
being built-in methods for each extended field type, use
`hasExtension(fieldName)` and `getExtension(fieldName)`.

The argument passed to `hasExtension()` and `getExtension()` is a
unique identifier for that field. The identifier is a static property of
the `NyctSubway` class. For example, the `nyct_trip_descriptor`
extended field has a unique identifier of
`NyctSubway.nyctTripDescriptor`.

Since this field is an extension to the standard `TripDescriptor`
field, you can check for its presence as follows:

```java
TripUpdate tripUpdate = entity.getTripUpdate();

TripDescriptor td = tripUpdate.getTrip();

if (td.hasExtension(NyctSubway.nyctTripDescriptor)) {
  // ...
}
```

The `NyctSubway.nyctTripDescriptor` identifier corresponds to a field
of the created class `NyctTripDesciptor`, meaning that you can
retrieve its value and assign it directly to the class type.

```java
NyctTripDescriptor nyctTd = td.getExtension(NyctSubway.nyctTripDescriptor);
```
You can now access the values directly from this instance of
`NyctTripDescriptor`. For example, one of the added fields is
direction, which indicates whether the general direction of the train is
North, South, East or West. The following code shows how to use this
value:

```java
if (nyctTd.hasDirection()) {
  Direction direction = nyctTd.getDirection();

  switch (direction.getNumber()) {
  case Direction.NORTH_VALUE: // Northbound train
    break;

  case Direction.SOUTH_VALUE: // Southbound train
    break;
  
  case Direction.EAST_VALUE: // Eastbound train
    break;
  
  case Direction.WEST_VALUE: // Westbound train
    break;

  }
}
```

Similarly, you can access other extended fields, either from the
`NyctTripDescriptor` object, or from an instance of the
`NyctStopTimeUpdate` extension.

### GTFS-realtime Extension Complete Example

Piecing together the snippets from this case study, the following code
shows in context how you can access the extra fields such as the
train's direction and identifier:

```java
public class NyctProcessor {

  // Loads and processes the feed
  public void process(String apiKey) throws IOException {

    URL url = new URL("http://datamine.mta.info/mta_esi.php?feed_id=1&key=" + apiKey);

    InputStream is = url.openStream();

    // Register the NYC-specific extensions

    ExtensionRegistry registry = ExtensionRegistry.newInstance();

    NyctSubway.registerAllExtensions(registry);

    FeedMessage fm = FeedMessage.parseFrom(is, registry);

    // Loop over all entities

    for (FeedEntity entity : fm.getEntityList()) {
      // In this example only trip updates are processed

      if (entity.hasTripUpdate()) {
        processTripUpdate(entity.getTripUpdate());
      }
    }
  }

  // Used to process a single trip update
  public void processTripUpdate(TripUpdate tripUpdate) {

    if (tripUpdate.hasTrip()) {

      TripDescriptor td = tripUpdate.getTrip();

      // Check if the extended trip descriptor is available

      if (td.hasExtension(NyctSubway.nyctTripDescriptor)) {
        NyctTripDescriptor nyctTd = td.getExtension(NyctSubway.nyctTripDescriptor);
        
        processNyctTripDescriptor(nyctTd);
      }
    }
  }

  // Process a single extended trip descriptor
  public void processNyctTripDescriptor(NyctTripDescriptor nyctTd) {

    // If the train ID is specified, output it
    if (nyctTd.hasTrainId()) {
      String trainId = nyctTd.getTrainId();

      System.out.println("Train ID: " + trainId);
    }

    // If the direction is specified, output it

    if (nyctTd.hasDirection()) {
      Direction direction = nyctTd.getDirection();

      String directionLabel = null;

      switch (direction.getNumber()) {
      case Direction.NORTH_VALUE:
        directionLabel = "North";
        break;

      case Direction.SOUTH_VALUE:
        directionLabel = "South";
        break;

      case Direction.EAST_VALUE:
        directionLabel = "East";
        break;
        
      case Direction.WEST_VALUE:
        directionLabel = "West";
        break;

      default:
        directionLabel = "Unknown Value";
      }

      System.out.println("Direction: " + directionLabel);
    }
  }
}
```

After you invoke the `process()` method, your output should be similar
to the following:

```
Direction: North

Train ID: 06 0139+ PEL/BBR
```
