---
name: sun-and-moon-times
description: Use when the user asks when the sun rises or sets, when it gets light or dark, about twilight, golden hour or blue hour, day length, or moonrise, moonset and moon phase for a place and date.
---

# Sun and moon times

The `sunrise-sunset` server answers these questions for any point on Earth and any date.

1. Turn the place into coordinates (decimal degrees). If the place is ambiguous, ask which one.
2. One date: call `get_sun_times` with `lat`, `lng` and `date` (`today`, `tomorrow` or `YYYY-MM-DD`).
3. Several consecutive dates (a trip, a month, a season): call `get_sun_times_range` once with `date_start` and `date_end`, up to 366 days, instead of one call per day.
4. Leave `tz` out: times come back in the local time of the place, already including daylight saving.
5. Read `sun_status`: on `midnight_sun` or `polar_night` days the events that do not happen are `null`. Say so instead of giving a time.
6. Golden hour and blue hour come in the morning and the evening; give the one the user asked about, or both.
7. When you show the times, mention that the data comes from Sunrise-Sunset.org.
