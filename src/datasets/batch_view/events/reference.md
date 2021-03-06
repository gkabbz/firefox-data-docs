# Events

This derived dataset makes it easier to analyze the Firefox [event ping][event_ping].
It has the following advantages over accessing the raw ping table (`telemetry.event`):

- There is no need to `UNNEST` the `events` column: this is already done for you.
- You don't have to know which process type emitted your event. If you care, you can query the `event_process` column.
- It is clustered on the `event_category` column, which can dramatically speed up your query.

[event_ping]: ../../pings.md#event-ping

## Data Reference

The events dataset contains one row for each event submitted in an event ping for that day.

The `category`, `method`, `object`, `value`, and `extra` fields of the event
are mapped to columns named `event_category`, `event_method`, `event_object`,
`event_string_value`, and `event_map_values`.
To access the `event_map_values`, you can use the `mozfun.map.get_key` UDF,
like `SELECT mozfun.map.get_key(event_map_values, "branch") AS branch FROM telemetry.events`.

### Example Query

This query gets the count of the number of times the user initiated the `dismiss_breach_alert`
and `learn_more_breach` actions. Note the use of the `event_category` to optimize the query:
for this example, this reduces the amount of data scanned from 450 GB to 52 MB.

```sql
SELECT countif(event_method = 'dismiss_breach_alert') AS n_dismissing_breach_alert,
       countif(event_method = 'learn_more_breach') AS n_learn_more
FROM telemetry.events_v1
WHERE event_category = 'pwmgr'
  AND submission_date='2020-04-20'
  AND sample_id=0
```

[link](https://sql.telemetry.mozilla.org/queries/73401/source)

## Scheduling

The events dataset is updated daily.
The job is scheduled on [Airflow](https://github.com/mozilla/telemetry-airflow).
The DAG is defined in [`dags/copy_deduplicate.py`](https://github.com/mozilla/telemetry-airflow/blob/master/dags/copy_deduplicate.py).

## Code Reference

This dataset is generated by [BigQuery ETL](https://github.com/mozilla/bigquery-etl/). The query that generates the dataset is [`sql/telemetry_derived/event_events_v1/query.sql`](https://github.com/mozilla/bigquery-etl/blob/master/sql/telemetry_derived/event_events_v1/query.sql).

## More Information

Firefox has an API to record events, which are then submitted through the main ping.
The format and mechanism of event collection in Firefox is documented [in the Firefox source documentation](https://firefox-source-docs.mozilla.org/toolkit/components/telemetry/telemetry/collection/events.html).

The full events data pipeline is [documented in the event pipeline documentation](../../../concepts/pipeline/event_pipeline.md).
