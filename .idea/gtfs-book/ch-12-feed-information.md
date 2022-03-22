## 12. Feed Information (feed_info.txt)

*This file is ***optional*** in a GTFS feed.*

Feed providers can include additional information about a feed using
`feed_info.txt`. It should only ever have a single row (other than the
CSV header row).

| Field  | Required? | Description |
| :----- | :-------- | :---------- |
| `feed_publisher_name` | Required | The name of the organization that publishes the feed. This may or may not be the same as any agency in `agency.txt`. |
| `feed_publisher_url` | Required | The URL of the feed publisher's web site. |
| `feed_lang` | Required | This specifies the language used in the feed. If an agency also has a language specified, then the agency's value should override this value. |
| `feed_start_date` | Optional | This value is a date in `YYYYMMDD` format that asserts the data in this feed is valid from this date. If specified, it typically matches up with the earliest date in `calendar.txt` or `calendar_dates.txt`, but if it is earlier, this is explicitly saying there are no services running between this date and the earliest service date. |
| `feed_end_date` | Optional | This value is a date in `YYYYMMDD` format that asserts the data in this feed is valid until this date. If specified, it typically matches up with the latest date in `calendar.txt` or `calendar_dates.txt`, but if it is earlier, this is explicitly saying there are no services running between the latest service date and this date. |
| `feed_version` | Optional | A string that indicates the version of this feed. This can be useful to let feed publishers know whether the latest version of their feed has been incorporated. |

### Sample Data

The following sample data is taken from the GTFS feed of TriMet in
Portland (<https://openmobilitydata.org/p/trimet>).

| `feed_publisher_name` | `feed_publisher_url`                    | `feed_lang` | `feed_start_date` | `feed_end_date` | `feed_version`    |
| :-------------------- | :-------------------------------------- | :---------- | :---------------- | :-------------- | :---------------- |
| TriMet                | [http://trimet.org](http://trimet.org/) | en          |                   |                 | 20140121-20140421 |

In this example, TriMet do not include the start or end dates, meaning
you should derive the dates this feed is active for by the dates in
`calendar_dates.txt` (this particular feed does not have a
`calendar.txt` file).

TriMet use date stamps to indicate the feed version. This feed was
published on 21 January 2014 and includes data up until 21 April 2014,
so it appears they use the first/last dates as a way to specify their
version. Each agency has its own method.

