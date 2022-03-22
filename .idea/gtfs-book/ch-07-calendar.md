## 7. Trip Schedules (calendar.txt & calendar_dates.txt)

*Each of these files are ***optional*** in a GTFS feed, but at least one
of them is ***required***.*

The `calendar.txt` file is used to indicate the range of dates on
which trips are running. It works by including a start date and a finish
date (typically a range of 3-6 months), then a marker for each day of
the week on which it operates. If there are single-day scheduling
changes that occur during this period, then the `calendar_dates.txt`
file can be used to override the schedule for each of these days.

The following table shows the specification for `calendar.txt`.

| Field  | Required? | Description |
| :----- | :-------- | :---------- |
| `service_id` | Required | A unique ID for a single service. This value is referenced by trips in `trips.txt`. |
| `start_date` | Required | This indicates the start date for a given service, in `YYYYMMDD` format. |
| `end_date` | Required | This indicates the end date for a given service, in `YYYYMMDD` format. |
| `monday` | Required | Contains `1` if trips run on Mondays between the start and end dates, `0` or empty if not. |
| `tuesday` | Required | Contains `1` if trips run on Tuesdays between the start and end dates, `0` or empty if not. |
| `wednesday` | Required | Contains `1` if trips run on Wednesdays between the start and end dates, `0` or empty if not. |
| `thursday` | Required | Contains `1` if trips run on Thursdays between the start and end dates, `0` or empty if not. |
| `friday` | Required | Contains `1` if trips run on Fridays between the start and end dates, `0` or empty if not. |
| `saturday` | Required | Contains `1` if trips run on Saturdays between the start and end dates, `0` or empty if not. |
| `sunday` | Required | Contains `1` if trips run on Sundays between the start and end dates, `0` or empty if not. |

As mentioned above, the `calendar_dates.txt` file is used to define
exceptions to entries in `calendar.txt`. For instance, if a 3-month
service is specified in `calendar.txt` and a holiday lies on a Monday
during this period, then you can use calendar_dates.txt to override this
single date.

If the weekend schedule were used for a holiday, then you would add a
record to remove the regular schedule for the holiday date, and another
record to add the weekend schedule for the holiday date.

Some feeds choose only to include `calendar_dates.txt` and not
`calendar.txt`, in which case there is an "add service" record for
every service and every date in this file.

The following table shows the specification for `calendar_dates.txt`.

| Field  | Required? | Description |
| :----- | :-------- | :---------- |
| `service_id` | Required | The service ID that an exception is being defined for. This is referenced in both `calendar.txt` and in `trips.txt`. Unlike `calendar.txt`, it is possible for a `service_id` value to appear multiple times in this file. |
| `date` | Required | The date for which the exception is occurring, in `YYYYMMDD` format. |
| `exception_type` | Required | This indicates whether the exception is denoting an added service (`1`) or a removed service (`2`). |

### Sample Data

The following is an extract from the `calendar.txt` file in Adelaide
Metro's GTFS feed (<https://openmobilitydata.org/p/adelaide-metro>). This
extract includes schedules from the start of 2014 until the end of March
2014.

| `service_id` | `monday` | `tuesday` | `wednesday` | `thursday` | `friday` | `saturday` | `sunday` | `start_date` | `end_date` |
| :----------- | :------- | :-------- | :---------- | :--------- | :------- | :--------- | :------- | :----------- | :--------- |
| 1            | 1        | 1         | 1           | 1          | 1        | 0          | 0        | 20140102     | 20140331   |
| 11           | 0        | 0         | 0           | 0          | 0        | 1          | 0        | 20140102     | 20140331   |
| 12           | 0        | 0         | 0           | 0          | 0        | 0          | 1        | 20140102     | 20140331   |

Any trip in the corresponding `trips.txt` file with a `service_id`
value of `1` runs from Monday to Friday. Trips with a `service_id`
of `11` run only on Saturday, while those with `12` run only on
Sunday.

Now consider an extract from `calendar_dates.txt` from the same feed,
as shown in the following table.

| `service_id` | `date`   | `exception_type` |
| ------------ | -------- | ---------------- |
| 1            | 20140127 | 2                |
| 1            | 20140310 | 2                |
| 12           | 20140127 | 1                |
| 12           | 20140310 | 1                |

The first two rows mean that on January 27 and March 10 trips with
`service_id` of `1` are not running. The final two rows mean that on
those same dates trips with `service_id` of `12` are running. This
has the following meaning:

**"On 27 January and 10 March, use the Sunday timetable instead of the
Monday-Friday timetable."**

In Adelaide, these two dates are holidays (Australia Day and Labour
Day). It is Adelaide Metro's policy to run their Sunday timetable on
public holidays, which is reflected by the above records in their
`calendar_dates.txt` file.

### Structuring Services

The case described above is the ideal case for specifying services in a
GTFS feed (dates primarily specified in `calendar.txt` with a handful
of exceptions in `calendar_dates.txt`).

Be aware that there are two other major ways that services are specified
in feeds.

1.  Using only `calendar_dates.txt` and expressly including every
    single date within the service range. Each of these is included as
    "service added" (an `exception_type` value of `1`). The
    following table shows how this might look.

| `service_id` | `date`   | `exception_type` |
| :----------- | :------- | :--------------- |
|  1           | 20140102 | 1                |
|  1           | 20140103 | 1                |
|  11          | 20140104 | 1                |
|  12          | 20140105 | 1                |

2.  Not using `calendar_dates.txt`, but creating many records in
    `calendar.txt` instead to span various dates. The following table
    shows how you can represent Monday-Friday from the sample data in
    this fashion.

| `service_id` | `monday` | `tuesday` | `wednesday` | `thursday` | `friday` | `saturday` | `sunday` | `start_date` | `end_date` |
| :----------- | :------- | :-------- | :---------- | :--------- | :------- | :--------- | :------- | :----------- | :--------- |
| 1a           | 1        | 1         | 1           | 1          | 1        | 0          | 0        | 20140102     | 20140126   |
| holiday1     | 1        | 0         | 0           | 0          | 0        | 0          | 0        | 20140127     | 20140127   |
| 1b           | 1        | 1         | 1           | 1          | 1        | 0          | 0        | 20140128     | 20140309   |
| holiday2     | 1        | 0         | 0           | 0          | 0        | 0          | 0        | 20140310     | 20140310   |
| 1c           | 1        | 1         | 1           | 1          | 1        | 0          | 0        | 20140311     | 20140331   |

In this example, each holiday has its own row in `calendar.txt` that
runs for a single day only.

Refer to *Finding Service IDs* to see how to determine
services that are running for a given day.

### Service Name

There are a number of feeds that specify a column in `calendar.txt`
called `service_name`. This is used to give a descriptive name to each
service. For example, the Sedona Roadrunner in Arizona
(<https://openmobilitydata.org/p/sedona-roadrunner>) has services called
"Weekday Service", "Weekend Service" and "New Year's Eve Service".

