## 6. Consuming Service Alerts

The previous chapter introduced you to Protocol Buffers and showed you
how load a remote GTFS-realtime feed into your Java project. This
chapter will show you how to read the data from each of the three entity
types (service alerts, vehicle positions and trip updates).

The previous chapter also showed you how to loop over all entities in a
feed using `getEntityList()`. Each entity contains either a service
alert, a vehicle position or a trip update.

Once you have verified that a `FeedEntity` element contains an alert,
you can retrieve the corresponding `Alert` object using
`getAlert()`.

```java
for (FeedEntity entity : fm.getEntityList()) {
	if (entity.hasAlert()) {
		Alert alert = entity.getAlert();

		processAlert(alert);
	}
}
```

You can then access the specific properties of a service alert using the
returned object.

### Cause & Effect

For example, to retrieve the cause value for the alert, you would first
check for its presence with `hasCause()` then retrieve the value using
`getCause()`.

```java
public void processAlert(Alert alert) {
	if (alert.hasCause()) {
		Cause cause = alert.getCause();

		// ...
	}

	// ...
}
```

The `Cause` object is an enumerator, meaning it has a finite number of
possible values. To determine which value the object corresponds to,
call `getNumber()` to compare it to the possible values.

```java
switch (cause.getNumber()) {
	case Cause.ACCIDENT_VALUE:
		// ...

	case Cause.MEDICAL_EMERGENCY_VALUE:
		// ...
}
```

Note: There are other possible cause values to include in the switch
statement; these have been omitted here as they are all covered in the
specification earlier in this book.

The `Effect` field works in the same way. The difference is that the
possible list of values to compare against is different.

```java
if (alert.hasEffect()) {
	Effect effect = alert.getEffect();

	switch (effect.getNumber()) {
	case Effect.DETOUR_VALUE:
		// ...

	case Effect.SIGNIFICANT_DELAYS_VALUE:
		// ...
	}
}
```

### Title, Description and URL

Each of these fields are of type `TranslatedString`. A
`TranslatedString` may contain multiple `Translation` objects, so
when processing these fields you must loop over the available
translations.

For example, to loop over the available translations for the header text
you iterate over **getTranslationList()**.

```java
if (alert.hasHeaderText()) {
	TranslatedString header = alert.getHeaderText();

	for (Translation translation : header.getTranslationList()) {
		// Process the translation here

	}
}
```

***Note:** To access the description you would use `hasDescription()` and
`getDescription()`, while to access the URL you would use `hasUrl()` and
`getUrl()`.*

Alternatively, you can use `getTranslationCount()` and
`getTranslation()` to retrieve each of the available translations.

```java
for (int i = 0; i < header.getTranslationCount(); i++) {
	Translation translation = header.getTranslation(i);

	// Process the translation here
}
```

A `Translation` object is made up of text and optionally, its
associated language. When dealing with the URL field, the text retrieved
from `getText()` contains a full URL.

```java
if (translation.hasLanguage()) {
	String language = translation.getLanguage();

	if (language.equals("fr")) {
		// Do something for French language
	}
	else {
		// All other languages
	}
}

if (translation.hasText()) {
	String text = translation.getText();

	// Do something with the text
}
```

***Note:** Most GTFS-realtime feeds only specify text in a single
language, and therefore do not include the language value.*

### Active Period

A service alert may contain zero or more time ranges, each of which
specify the dates and times the alert is active for. If none are
specified then the alert is active as long as it exists within the feed.

You can access each of the `TimeRange` objects using either of the
following methods:

```java
for (TimeRange timeRange : alert.getActivePeriodList()) {
	// ...
}

for (int i = 0; i < alert.getActivePeriodCount(); i++) {
	TimeRange timeRange = alert.getActivePeriod(i);

	// ...
}
```

A `TimeRange` can have either a start or finish date, or it may
contain both. In Java, you can turn each of these dates into a
`java.util.Date` object as shown below.

```java
if (timeRange.hasStart()) {
	Date start = new Date(timeRange.getStart() * 1000);

	// ...
}

if (timeRange.hasEnd()) {
	Date end = new Date(timeRange.getEnd() * 1000);

	// ...
}
```

***Note:** The date value is multiplied by 1,000 because the date in the
GTFS-realtime is represented by the number of seconds since January 1,
1970, while `java.util.Date` is instantiated using the number of
milliseconds since the same date.*

### Affected Entities

A service alert may contain zero or more affected entities, each of
which describes a route, stop, agency, trip or route type. You can
access these entities using either of the following methods:

```
for (EntitySelector entity : alert.getInformedEntityList()) {

}

for (int i = 0; i < alert.getInformedEntityCount(); i++) {
	EntitySelector entity = alert.getInformedEntity(i);

}
```

There are a number of properties available in the `EntitySelector`
object, each of which can be used to match the entity to the
corresponding GTFS feed.

For example, if the `EntitySelector` object has a route ID value, then
you should be able to locate the route in the corresponding GTFS feed's
`routes.txt` file.

The properties can be accessed as follows:

```java
if (entity.hasAgencyId()) {
	String agencyId = entity.getAgencyId();

}

if (entity.hasRouteId()) {
	String routeId = entity.getRouteId();

}

if (entity.hasRouteType()) {
	int routeType = entity.getRouteType();

}

if (entity.hasStopId()) {
	String stopId = entity.getStopId();

}
```

The route type value is an Integer and if present must correspond either
to the standard GTFS route type values, or to the extended route type
values.

The other entity information that can be contained in `EntitySelector`
is trip information. You can access the trip properties as follows:

```java
if (entity.hasTrip()) {
	TripDescriptor trip = entity.getTrip();

	if (trip.hasTripId()) {
		String tripId = trip.getTripId();

	}

	if (trip.hasRouteId()) {
		String routeId = trip.getRouteId();
		
	}

	if (trip.hasStartDate()) {
		String startDate = trip.getStartDate();

	}

	if (trip.hasStartTime()) {
		String startTime = trip.getStartTime();

	}

	if (trip.hasScheduleRelationship()) {
		ScheduleRelationship sr = trip.getScheduleRelationship();

	}
}
```

You can test the `ScheduleRelationship` value by comparing the
`getNumber()` value to one of the available constants, as follows.

```java
if (entity.hasTrip()) {
	// ...

	if (trip.hasScheduleRelationship()) {

		ScheduleRelationship sr = trip.getScheduleRelationship();

		switch (sr.getNumber()) {
			case ScheduleRelationship.ADDED_VALUE:
				// ...
				break;

			case ScheduleRelationship.CANCELED_VALUE:
				// ...
				break;

			case ScheduleRelationship.SCHEDULED_VALUE:
				// ...
				break;

			case ScheduleRelationship.UNSCHEDULED_VALUE:
				// ...
				break;
		}
	}
}
```
