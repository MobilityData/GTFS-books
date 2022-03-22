# The Definitive Guide to GTFS-realtime

*How to consume and produce real-time public transportation data with the GTFS-rt specification.*

Originally written by Quentin Zervaas.

## About This Book

This book is a comprehensive guide to GTFS-realtime, a specification for
publishing of real-time public transportation data. GTFS-realtime is
designed to complement the scheduled data that hundreds of transit
agencies around the world publish using GTFS (General Transit Feed
Specification).

This book begins with a description of the specification, with
discussion about the three types of data contained in GTFS-realtime
feeds: service alerts; vehicle positions; and trip updates.

Next the reader is introduced to *Protocol Buffers*, the data format
that GTFS-realtime uses when it is being transmitted. This section then
instructs the reader how to consume the three types of data from
GTFS-realtime feeds (both from standard feeds and feeds with
*extensions*).

Finally, the reader is shown how to produce a GTFS-realtime feed. A
number of examples in this book use Java, but the lessons can be applied
to a number of different languages.

This book complements *The Definitive Guide to GTFS*.

### About The Author

Quentin was the founder of TransitFeeds (now <OpenMobilityData.org>), a web
site that provides a comprehensive listing of public transportation data
available around the world. This site is referenced various times
throughout this book.

### Credits

First Edition. Published in August 2015.

**Technical Reviewer**

Nick Maher

**Copy Editors**

Anne Zervaas
Miranda Little

**Disclaimer**

The information in this book is distributed on an "as is" basis, without
warranty. Although every precaution has been taken in the preparation of
this work, the author shall not be liable to any person or entity with
respect to any loss or damage caused or alleged to be caused directly or
indirectly by the information contained in this book.

