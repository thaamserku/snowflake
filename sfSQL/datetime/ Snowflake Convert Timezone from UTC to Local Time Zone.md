---
created: 2023-09-25T20:14:33 (UTC -04:00)
tags: []
source: https://popsql.com/learn-sql/snowflake/how-to-convert-utc-to-local-time-zone-in-snowflake
author: 
---

# Snowflake: Convert Timezone from UTC to Local Time Zone 

When storing timestamps, Snowflake stores time zone data in the form of adding the offset at the end of the timestamp. That offset code tells us the time zone of timestamps.

Snowflake uses the host server time as the basis for generating the output of `current_timestamp()`.

Default timezone in Snowflake is Pacific Daylight Time (PDT). To convert a PDT timestamp to a UTC or a local time zone, you can use the following:

```sql
select
  current_timestamp() as pdt_time_zone,
  convert_timezone('UTC', current_timestamp()) as utc_time_zone,
  convert_timezone('Europe/Paris', current_timestamp()) as europe_paris_time_zone
```

> _Note: Snowflake supports [IANA](https://www.iana.org/time-zones) time zone database._
