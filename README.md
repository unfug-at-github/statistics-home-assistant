This is an enhanced copy of the home assistant statistics component including some improvements.

It fixes several issues:
- **Spikes at startup** due to a lack of synchronization of the internal value queue 
- Values are provided even if only a **single value** is in the buffer (the standard component returns unknown in these cases). E.g. the average of [7] becomes 7, not unknown.
- When a time interval is specified the average is computed over the given interval. The standard component only computes the average between the first and the last value change within the interval, which ignores the time before the first change as well as the time after the last change. This way of comuting time-based average leads to results that have led to a lot of questions / discussions in the forum.
- The value before the first value change within the interval is taken into account for other computations as well. E.g. if the value changes only once from 2.0 to 1.0 within the interval, the max would be computed as 1.0 by the standard component. Now it becomes 2.0 (the last value reported before the interval).

It also adds one additional feature:
- A "refresh_interval" parameter allows to re-compute the value on a regular basis. Usually values only change when a new value is received or an old value expires. However, the value of time-based statistics change just because the interval is shifting with time, and so a re-calculation even without external triggers is making sense.

As a side effect, the "keep_last_sample" will have no effect if "max_age" was specified

It also includes a test setup (configuration.yaml) to demonstrate the changes. You can easily see how the ouput of the standard and the custom component differ. The yellow line is the input, the blue line the standard sensor and the orange one this custom sensor. The blue line is broken in several places because the standard sensor is unable to compute a value if there are less than two value changes in the buffer.

![screenshot](screenshot.png)

Here is the definition of the sample sensors:
```
sensor: 
  - platform: statistics_custom
    name: "xxx_test_avg_step_10s_refresh"
    entity_id: sensor.xxx_test_float
    state_characteristic: average_step
    max_age:
      seconds: 10
    refresh_interval:
      seconds: 1
  
  - platform: statistics_custom
    name: "xxx_test_avg_linear_10s_refresh"
    entity_id: sensor.xxx_test_float
    state_characteristic: average_linear
    max_age:
      seconds: 10
    refresh_interval:
      seconds: 1
```
I managed to get some of the changes integrated into the core system, but not all of them.
The following changes were added to the core system, so they should be included in one of the next releases:

- spikes
- single value issue
- adding values to the buffer even when the value wasn't changed (has a similar effect like the refresh interval)

It doesn't seem like I will get the remaining things in, so I leave this here as an alternative for those who want the other features as well.

This enhanced component solves issues discussed here:
[1](https://github.com/home-assistant/core/issues/119738)
[2](https://github.com/home-assistant/core/issues/98262)
[3](https://github.com/home-assistant/core/issues/67627)

Here is an image of the two time-based averages (linear / step) following an input curve:
![average_image](https://private-user-images.githubusercontent.com/65363098/362150611-0c67a119-5b83-467e-a51d-82203b7f2e21.png?jwt=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJnaXRodWIuY29tIiwiYXVkIjoicmF3LmdpdGh1YnVzZXJjb250ZW50LmNvbSIsImtleSI6ImtleTUiLCJleHAiOjE3Mjk0MTY1MzAsIm5iZiI6MTcyOTQxNjIzMCwicGF0aCI6Ii82NTM2MzA5OC8zNjIxNTA2MTEtMGM2N2ExMTktNWI4My00NjdlLWE1MWQtODIyMDNiN2YyZTIxLnBuZz9YLUFtei1BbGdvcml0aG09QVdTNC1ITUFDLVNIQTI1NiZYLUFtei1DcmVkZW50aWFsPUFLSUFWQ09EWUxTQTUzUFFLNFpBJTJGMjAyNDEwMjAlMkZ1cy1lYXN0LTElMkZzMyUyRmF3czRfcmVxdWVzdCZYLUFtei1EYXRlPTIwMjQxMDIwVDA5MjM1MFomWC1BbXotRXhwaXJlcz0zMDAmWC1BbXotU2lnbmF0dXJlPTY2MGRhYTQ3MTViODZlZjVmMDAyMWMxOGUyYWI2M2NhZGFkMTUyNzk2YmNiYTgyODI2NTQzMTBjZGRmNWJmODMmWC1BbXotU2lnbmVkSGVhZGVycz1ob3N0In0.HI8QdIptVbquvt6lVslNoZNWQrG4Aq1g0MY95Ph-qIo)

Please refer to [homeassistant core](https://github.com/unfug-at-github/home-assistant-core) regarding topics around copyright etc.
