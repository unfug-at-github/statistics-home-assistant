This is a copy of the home assistant statistics component including some improvements.

It fixes several issues:
- Spikes at startup
- Incorrect values due to a lack of synchronization of the internal value queue
- Values are provided even if only a single value is in the buffer (the standard component returns unknown in these cases). E.g. the average of [7] becomes 7, not unknown.
- When a time interval is specified the average is computed over the given interval. The standard component only computes the average between the first and the last value change within the interval, which ignores the time before the first change as well as the time after the last change. This way of comuting time-based average leads to results that have led to a lot of questions / discussions in the forum.
- The value before the first value change within the interval is taken into account for other computations as well. E.g. if the value changes only once from 2.0 to 1.0 within the interval, the max would be computed as 1.0 by the standard component. Now it becomes 2.0 (the last value reported before the interval).

It also adds one additional feature:
- A "refresh_interval" parameter allows to re-compute the value on a regular basis. Usually values only change when a new value is received or an old value expires. However, the value of time-based statistics change just because the interval is shifting with time, and so a re-calculation even without external triggers is making sense.
