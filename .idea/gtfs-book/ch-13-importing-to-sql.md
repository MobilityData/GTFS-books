## 13. Importing a GTFS Feed to SQL

One of the great things about GTFS is that it is already in a format
conducive to being used in an SQL database. The presence of various IDs
in each of the different files makes it easy to join the tables in order
to extract the data you require.

To try this yourself, download `GtfsToSql`
(<https://github.com/OpenMobilityData/GtfsToSql>). This is a Java
command-line application that imports a GTFS feed to an SQLite database.
This application also supports PostgreSQL, but the examples used here
are for SQLite.

The pre-compiled `GtfsToSql` Java archive can be downloaded from its
GitHub repository at
<https://github.com/OpenMobilityData/GtfsToSql/tree/master/dist>.

To use `GtfsToSql`, all you need is an extracted GTFS feed. The
following instructions demonstrate how you to import the TriMet feed
that has been referenced throughout this book.

Firstly, download and extract the feed. The following commands use curl
to download the file, then unzip to extract the file to a sub-directory
called `trimet`.

```
$ curl http://developer.trimet.org/schedule/gtfs.zip > gtfs.zip

$ unzip gtfs.zip -d trimet/
```

To create an SQLite database from this feed, the following command can
be used.

```
$ java -jar GtfsToSql.jar -s jdbc:sqlite:./db.sqlite -g ./trimet
```

This may take a minute or two to complete (you will see progress as it
imports the feed and then creates indexes), and at the end you will have
a GTFS database in a file called `db.sqlite`. You can then query this
database with the command-line `sqlite3` tool, as shown in the
following example.

```
$ sqlite3 db.sqlite
sqlite> SELECT * FROM agency;
|TriMet|http://trimet.org|America/Los_Angeles|en|503-238-7433

sqlite> SELECT * FROM routes WHERE route_type = 0;
90|||MAX Red Line||0|http://trimet.org/schedules/r090.htm||
100|||MAX Blue Line||0|http://trimet.org/schedules/r100.htm||
190|||MAX Yellow Line||0|http://trimet.org/schedules/r190.htm||
193||Portland Streetcar|NS Line||0|http://trimet.org/schedules/r193.htm||
194||Portland Streetcar|CL Line||0|http://trimet.org/schedules/r194.htm||
200|||MAX Green Line||0|http://trimet.org/schedules/r200.htm||
250|||Vintage Trolley||0|http://trimet.org/schedules/r250.htm||
```

The first query above finds all agencies stored in the database, while
the second finds all routes marked as Light Rail (`route_type` of
`0`).

**Note: In the following chapters there are more SQL examples. All of
these examples are geared towards running on an SQLite database that has
been created in this manner.**

All tables in this database match up with the corresponding GTFS
filename (so for `agency.txt`, the table name is `agency`, while for
`stop_times.txt` the table name is `stop_times`). The columns in SQL
have the same name as the value in the corresponding GTFS file.

***Note:** All data imported using this tool is stored as text in the
database. This means you may need to be careful when querying integer
data. For example, ordering stop times by stop_sequence may not produce
expected results (for instance, 29 as a string comes before 3). Although
it is a performance hit, you can change this behavior by casting the
value to integer, such as: ORDER BY stop_sequence + 0. The reason
`GtfsToSql` works in this way is because it is intended as a lightweight
tool to be able to quickly query GTFS data. I recommend rolling your own
importer to treat data exactly as you need it, especially in conjunction
with some of the optimization techniques recommended later in this book.*

### File Encodings

The GTFS specification does not indicate whether files should be encoded
using UTF-8, ISO-8859-1 or otherwise. Since a GTFS feed is not
necessarily in English, you must be willing to handle an extended
character set.

The GtfsToSql tool introduced above automatically detects the encoding
of each file using the juniversalchardet Java library
(<https://code.google.com/p/juniversalchardet/>).

I recommend you take some time looking at the source code of GtfsToSql
to further understand this so you are aware of handling encodings
correctly if you write your own parser.

### Optimizing GTFS Feeds

If you are creating a database that is to be distributed onto a mobile
device such as an iPhone or Android phone, then disk space and
computational power is at a premium. Even if you are setting up a
database to be queried on a server only, then making the database
perform as quickly as possible is still important.

In the following chapters are techniques for optimizing GTFS feeds.
There are many techniques that can be applied to improve the performance
of GTFS, such as:

* Using integer identifiers rather than string identifiers (for route
  IDs, trip IDs, stop IDs, etc.) and creating appropriate indexes
* Removing redundant shape points and encoding shapes
* Deleting unused data
* Reusing repeating trip patterns.

Changing the data to use integer IDs makes the greatest improvement to
performance, but the other techniques also help significantly.

Depending on your needs, there are other optimizations that can be made
to reduce file size and speed up querying of the data, but the ease of
implementing them may depend on your database storage system and the
programming language used to query the data. The above list is a good
starting point.

