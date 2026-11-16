# Conventions, time correction and known limits

## Selectable conventions

Practitioners disagree on two boundaries. Both are parameters. Every response states which was applied in `conventions`.

| Tool | Parameter | Default | Alternative | Affects |
|---|---|---|---|---|
| `cast_bazi` | `day_boundary` | `zi_hour_23`: a birth at 23:00–23:59 true solar time takes the next day's pillar | `midnight_00`: it keeps the same day's pillar | The day pillar for births in the last hour of the day |
| `cast_ziwei` | `year_boundary` | `lunar_new_year`: the year stem-branch changes on the first day of the first lunar month | `lichun`: it changes at the Start of Spring solar term | Births between Lunar New Year and Start of Spring |

The day-boundary test is made on true solar time, not on clock time.

## How time is corrected

**BaZi.** Clock time is converted to true solar time using the birthplace longitude, the equation of time and the daylight-saving rules of the birthplace time zone, including China's 1986–1991 daylight-saving years. `chart.solar_time_info` records each step: `longitude_correction`, `equation_of_time`, `dst_applied`, `total_offset_minutes`, `true_solar_time`, `original_shichen`, `corrected_shichen`, `shichen_changed`.

**Zi Wei.** For birthplaces in `Asia/Shanghai` the correction is longitude only: no equation of time and no daylight saving. For every other zone the chart uses full true solar time. `time_basis` records what was applied.

## Known limits

- During China's 1986–1991 daylight-saving periods, the BaZi and Zi Wei tools can work from times about an hour apart for the same `Asia/Shanghai` birth, for the reason above. Say so if the user compares the two.
- For births on Kiritimati, in Tonga, Samoa or the Chatham Islands, the solar date used can be one day earlier than the local calendar date.
- Overlapping palaces (叠宫) are not returned. `chart.diegong` reports the field as held.
- Charts are cast only from an exact birth time. A date alone or an approximate time is not accepted.
- `source.url` opens BaziChart's chart page, not the specific chart just calculated.
- Birth years 1900–2100 only.
