## 1. Introduction to GTFS

GTFS (General Transit Feed Specification) is a data standard developed
by Google used to describe a public transportation system. Its primary
purpose was to enable public transit agencies to upload their schedules
to Google Transit so that users of Google Maps could easily figure out
which bus, train, ferry or otherwise to catch.

> **A GTFS feed is a ZIP file that contains a series of CSV files that
list routes, stops and trips in a public transportation system.**

This book examines GTFS in detail, including which data from a public
transportation system can be represented, how to extract data, and
explores some more advanced techniques for optimizing and querying data.

The official GTFS specification has been referenced a number of times in
this book. It is strongly recommended you are familiar with it. You can
view at the following URL:

<https://developers.google.com/transit/gtfs/reference>

### Structure of a GTFS Feed

A GTFS feed is a series of CSV files, which means that it is trivial to
include additional files in a feed. Additionally, files required as part
of the specification can also include additional columns. For this
reason, feeds from different agencies generally include different levels
of detail.

***Note:** The files in a GTFS feed are CSV files, but use a file
extension of `.txt`.*

A GTFS feed can be described as follows:

> **A GTFS feed has one or more routes. Each route (`routes.txt`) has one or
more trips (`trips.txt`). Each trip visits a series of stops (`stops.txt`)
at specified times (`stop_times.txt`). Trips and stop times only contain
time of day information; the calendar is used to determine on which days
a given trip runs (`calendar.txt` and `calendar_dates.txt`).**

The following chapters cover the main files that are included in all
GTFS feeds. For each file, the main columns are covered, as well as
optional columns that can be included. This book also covers some of the
unofficial columns that some agencies choose to include.

