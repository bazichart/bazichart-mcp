# Parameters and response fields

## `cast_bazi`

| Parameter | Required | Notes |
|---|---|---|
| `year`, `month`, `day` | yes | Gregorian. Year 1900–2100 |
| `hour` | yes | 0–23, local clock time |
| `minute` | no | 0–59, default `0` |
| `gender` | yes | `male` / `female`, `M` / `F`, `男` / `女` |
| `longitude` | yes | −180 to 180, east positive |
| `timezone` | yes | IANA name |
| `day_boundary` | no | `zi_hour_23` (default) or `midnight_00` |
| `language` | no | `en` (default), `zh-CN`, `zh-TW`. Selects the language of `source.url` |

| Response field | Content |
|---|---|
| `chart.four_pillars` | `year`, `month`, `day`, `hour`, each with `heavenly_stem` and `earthly_branch` |
| `chart.ten_gods` | Ten god of each stem and hidden stem, relative to the day master |
| `chart.hidden_stems` | Hidden stems of the four branches |
| `chart.nayin` | Na Yin of the four pillars |
| `chart.dizhi_relations` | Combinations, clashes, punishments and harms among the branches |
| `chart.dayun` | Ten luck cycles: stem, branch, age range, exact start age, transition date-time |
| `chart.solar_time_info` | The time-correction record |
| `conventions.day_boundary` | The day boundary applied |
| `source` | `name`, `url`, `attribution` |

## `cast_ziwei`

| Parameter | Required | Notes |
|---|---|---|
| `year`, `month`, `day` | yes | Gregorian. Year 1900–2100 |
| `hour` | yes | 0–23, local clock time |
| `minute` | no | 0–59, default `0` |
| `gender` | yes | as above |
| `longitude` | yes | −180 to 180, east positive |
| `timezone` | no | IANA name, default `Asia/Shanghai`. Always pass the birthplace's zone |
| `year_boundary` | no | `lunar_new_year` (default) or `lichun` |
| `language` | no | as above |

| Response field | Content |
|---|---|
| `chart.ming_gong`, `chart.shen_gong` | Life Palace and Body Palace |
| `chart.wu_xing_ju` | Five-element bureau |
| `chart.year_ganzhi` | Year stem-branch under the selected `year_boundary` |
| `chart.ming_zhu`, `chart.shen_zhu` | Life ruler and body ruler stars |
| `chart.palaces` | Twelve palaces: branch, stem, name, `major_stars`, `major_star_brightness`, `auxiliary_stars`, `malefic_stars`, `sihua`, `is_shen_gong` |
| `chart.sihua` | Four Transformations of the birth-year stem |
| `chart.dalim` | Direction, start age and the ten-year decades |
| `time_basis` | Clock time, correction in minutes and the rule applied |
| `conventions.year_boundary` | The year boundary applied |
| `source` | `name`, `url`, `attribution` |

All chart terms are returned in Simplified Chinese. `chart.palaces` is an object keyed `"1"`–`"12"`, and the keys of `chart.dizhi_relations` are Chinese relationship names.

Two further parameters, `precision` and `shichen_index`, exist on both tools and should be left unset.
