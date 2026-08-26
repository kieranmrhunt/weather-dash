# Reading weather dashboard

This repository contains the static dashboard application published by GitHub
Pages. Live observations are deliberately not committed to Git.

## Data flow

1. `~/weather_dash_update.py` reads the Reading observatory feeds and atomically
   writes a versioned JSON payload into Reading's mounted public web space.
2. JASMIN's `xfer-vm-03` transfer cron mirrors that JSON into
   `~/incompass/public/kieran/weather-dash/latest.json` two minutes later.
3. `index.html` polls both public mirrors every minute, uses the fresher valid
   payload, and retains its embedded observation as an offline fallback.

Public endpoints:

- Dashboard: <https://kieranmrhunt.github.io/weather-dash/>
- Live data: <https://gws-access.jasmin.ac.uk/public/incompass/kieran/weather-dash/latest.json>
- Reading mirror: <https://www.met.reading.ac.uk/~rz908899/weather-dash/latest.php>

## Operations

Run and publish the live-data update immediately:

```bash
WEATHER_DASH_SINGLE_RUN=1 ~/run_weather_dash_update.sh --debug
```

Build and validate the JSON without uploading it:

```bash
WEATHER_DASH_SINGLE_RUN=1 ~/run_weather_dash_update.sh --no-upload --debug
```

The scheduled wrapper runs only on ten-minute boundaries. It accepts these
environment overrides:

```text
WEATHER_DASH_JSON_OUTPUT (defaults to Reading's mounted public directory)
WEATHER_DASH_PUBLISH_HOST
WEATHER_DASH_PUBLISH_PATH
WEATHER_DASH_SSH_KEY
WEATHER_DASH_DISABLE_UPLOAD=1
```

`WEATHER_DASH_PUBLISH_HOST` is empty by default because the normal scheduled
path needs no unattended SSH credential. Set it only for an interactive manual
rsync override.

## Updating the application

Rebuild `index.html` only when the template or dashboard code changes:

```bash
python3 ~/weather_dash_update.py --repo ~/weather-dash --no-git --debug
git -C ~/weather-dash diff -- index.html
```

After review, commit and push that application change normally. Routine
observation updates should never modify this repository.
