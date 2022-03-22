## 11. Publishing GTFS-realtime Feeds

So far this book has been focused on how to consume GTFS-realtime feeds;
in this chapter you will be shown how to create and publish your own
GTFS-realtime feeds.

While this chapter is primarily intended for transit agencies (or
third-party companies providing services to public transit companies),
this information can be useful in other situations also.

Even if you do not represent a transit agency or have access to the GPS
units of an entire bus fleet, there may still be situations where you
want to produce a GTFS feed. For example, if you have a trip planning
server that can only handle GTFS and GTFS-realtime data, you might build
your own GTFS-realtime feeds in the following situations:

* A transit company offers service alerts only via Twitter or an RSS
  feed.
* You can access vehicle positions or estimated arrivals from a feed
  in a format such as SIRI, NextBus or BusTime.
* You have interpolated your own vehicle positions based on
  GTFS-realtime trip updates.
* You have interpolated your own trip updates based on vehicle
  positions.

### Building Protocol Buffer Elements

When you generate source files using the `protoc` command, there is a
*builder* class created for each element type. To create an element to
include in a protocol buffer, you use its builder to construct the
element.

For example, a service alert entity uses the `Alert` class. To
construct your own service alert, you would use the `Alert.Builder`
class. The `Alert` class contains a static method called
`newBuilder()` to create an instance of `Alert.Builder`.

```java
Alert.Builder alert = Alert.newBuilder();
```

You can now set the various elements that describe a service alert.

```java
alert.setCause(Cause.ACCIDENT);

alert.setEffect(Effect.DETOUR);
```

Most elements will be more complex than this; you will need to build
them in a similar manner before adding them to the alert. For example,
the header text for a service alert uses the `TranslatedString`
element type, which contains one or more translations of a single
string.

```java
Translation.Builder translation = Translation.newBuilder();

translation.setText("Car accident");

TranslatedString.Builder translatedString = TranslatedString.newBuilder();

translatedString.addTranslation(translation);

alert.setHeaderText(translatedString);
```

In actual fact, you can chain together these calls, since the builder
methods return the builder. The first two lines of the above code can be
shortened as follows:

```java
Translation.Builder translation = Translation.newBuilder().setText("Car accident");
```

For repeating elements (such as the `informed_entity` field), use the
`addElementName()` method. In the case of `informed_entity`,
this would be `addInformedEntity()`. The following code adds an
informed entity to the alert for a route with an ID of 102:

```java
EntitySelector.Builder entity = EntitySelector.newBuilder().setRouteId("102");

alert.addInformedEntity(entity);
```

### Creating a Complete Protocol Buffer

The previous section showed the basics of creating a service alert
message, but a protocol buffer feed has more to it than just a single
entity. It can have multiple entities, and you must also include the
GTFS-realtime header. The header can be created as follows:

```java
FeedHeader.Builder header = FeedHeader.newBuilder();

header.setGtfsRealtimeVersion("1.0");
```

A single service alert (or a trip update, or a vehicle position) is
contained within a `FeedEntity` object. Each `FeedEntity` in a feed
must have a unique ID. The following code creates the `FeedEntity`
using the `alert` object created in the previous section.

```java
FeedEntity.Builder entity = FeedEntity.newBuilder();

entity.setId("SOME UNIQUE ID");

entity.setAlert(alert);
```

Once you have the header and an entity you can create the feed as
follows:

```java
FeedMessage.Builder message = FeedMessage.newBuilder();

message.setHeader(header);

message.addEntity(entity);
```

***Note**: A feed with no entities is also valid; in the middle of the night
there may be no vehicle positions or trip updates, and there may
frequently be no service alerts.*

Once you have created this object, you can turn it into a
`FeedMessage` by calling `build()`.

```
FeedMessage feed = message.build();
```

This will give you a `FeedMessage` object just like when you parse a
third-party feed using `FeedMessage.parseFrom()`.

### Full Source Code

Piecing together all of the code covered so far in this chapter, you
could create a service alert feed (using a fictional detour) using the
following code.

This example makes use of a helper method to build translated strings,
since it needs to be done a number of times. If you want to create the
alert in multiple languages, you would need to change this method
accordingly.

```java
public class SampleServicesAlertsFeedCreator {
  // Helper method to simplify creation of translated strings
  
  private TranslatedString translatedString(String str) {
    Translation.Builder translation = Translation.newBuilder().setText(str);
    
    return TranslatedString.newBuilder().addTranslation(translation).build();  
  }
  
  public FeedMessage create() {
    Alert.Builder alert = Alert.newBuilder();
    
    alert.setCause(Cause.ACCIDENT);
    alert.setEffect(Effect.DETOUR);
    alert.setUrl(translatedString("http://www.example.com"));
    alert.setHeaderText(translatedString("Car accident on 14th Street"));
    
    alert.setDescriptionText(translatedString(
      "Please be aware that 14th Street is closed due to a car accident"
    ));
    
    // Loop over several route IDs to mark them as impacted
    
    String impactedRouteIds[] = { "102", "103" };
    
    for (int i = 0; i < impactedRouteIds.length; i++) {
      EntitySelector.Builder entity = EntitySelector.newBuilder();
    
      entity.setRouteId(impactedRouteIds[i]);
    
      alert.addInformedEntity(entity);
    }
    
    // Create the alert container entity
    
    FeedEntity.Builder entity = FeedEntity.newBuilder();
    
    entity.setId("1");
    entity.setAlert(alert);
    
    // Build the feed header
    
    FeedHeader.Builder header = FeedHeader.newBuilder();
    
    header.setGtfsRealtimeVersion("1.0");
    
    // Build the feed using the header and entity
    
    FeedMessage.Builder message = FeedMessage.newBuilder();
    
    message.setHeader(header);
    message.addEntity(entity);
    
    // Return the built FeedMessage
    return message.build();
  }
}
```

### Modifying an Existing Protocol Buffer

In some circumstances you might want to modify an existing protocol
buffer. For example, consider a case where you have access to a service
alerts feed, but also want to add additional service alerts that you
parsed from Twitter. The following diagram demonstrates this:

![Modified Feed](images/modified-feed.png)

In this case, you can turn a `FeedMessage` object into a
`FeedMessage.Builder` object by calling to the `toBuilder()`` method.
You can then add additional alerts as required and create a new feed.

```java
// Parse some third-party feed

URL url = new URL("http://example.com/alerts.pb");

InputStream is = url.openStream();

FeedMessage message = GtfsRealtime.FeedMessage.parseFrom(is);

// Convert existing feed into a builder

FeedMessage.Builder builder = message.toBuilder();

Alert.Builder alert = Alert.newBuilder();

// Add the details of the alert here

// Create the alert entity

FeedEntity.Builder entity = FeedEntity.newBuilder();

entity.setId("SOME ID");
entity.setAlert(alert);

// Add the new entity to the builder
builder.addEntity(entity);

// Build the update the FeedMessage
message = builder.build();
```

### Saving a Protocol Buffer File

Once you have created a protocol buffer, the next step is to output it
so other systems that read GTFS-realtime feeds can consume it (such as
for others who publish real-time data in their apps or web sites).

Typically, you would generate a new version of the feed every `X`
seconds, then save (or upload) it each time to your web server (see the
next section for discussion on frequency of updates).

The raw protocol buffer bytes can be output using the `writeTo()`
method on the `FeedMessage` object. This method accepts an
`OutputStream` object as its only argument.

For example, to output the service alerts feed created in this chapter
to a file, you can use the `FileOutputStream` class.

Note: While there are no specific rules for naming a protocol buffer,
often the `.pb` extension is used.

```java
SampleServicesAlertsFeedCreator creator = new SampleServicesAlertsFeedCreator();

FeedMessage message = creator.create();

File file = new File("/path/to/output/alerts.pb");

OutputStream outputStream = new FileOutputStream(file);

message.writeTo(outputStream);
```

### Serving a Protocol Buffer File

The recommended content type header value to use when serving a Protocol
Buffer file is **application/octet-stream**. In Apache HTTP Server, you
can set the following configuration parameter to serve `.pb` files
with this content type:

```
AddType application/octet-stream .pb
```

If you are using nginx for your web server, you can add the following
entry to the nginx `mime.types` file:

```
types {
  ...

  application/octet-stream pb;
}
```

### Frequency of Updates

When publishing your own feed, the frequency in which you update the
feed on your web server depends on how frequently the source data is
updated.

It is important to take into account the capabilities of your servers
when providing a GTFS-realtime feed, as the more frequently the data is
updated, the more resources that are required. Each of the GTFS-realtime
message types has slightly different needs:

* **Vehicle Positions.** These will need updating very frequently, as
  presumably the vehicles on your transit network are always moving. A
  vehicle position feed could update as frequently as every 10-15
  seconds.
* **Trip Updates.** These will need updating very frequently, although
  perhaps not as frequently as vehicle positions. Estimates would
  constantly be refined by new vehicle positions, but a single
  movement (or lack of movement) for a vehicle is not likely to make a
  huge difference to estimates. A trip updates feed could update every
  10-30 seconds.
* **Service Alerts.** These will typically change far less frequently
  then vehicle positions or trip updates. A system that triggered an
  update to the service alerts feed only when a new alert was entered
  into the system would be far more efficient than automatically doing
  it every `X` seconds.

To summarize:

-   **Vehicle Positions.** Update every 10-15 seconds.
-   **Trip Updates.** Update every 10-30 seconds.
-   **Service Alerts.** Triggered on demand when new data is available.

If your transit agency does not run all night, an additional efficiency
would be to not update the feed at all when the network has shut down
for the night.

In this case, once the last trip has finished, an empty protocol buffer
would be uploaded (that is, a valid buffer but with no entities), and
the next version would not be uploaded until the next morning when the
first trip starts.

