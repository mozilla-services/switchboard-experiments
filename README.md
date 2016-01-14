# switchboard-experiments
![travis-ci](https://travis-ci.org/mozilla-services/switchboard-experiments.svg?branch=master)

This repository contains the JSON file that configures switchboard experiments, `experiments.json`.

The format of this file is as follows:

```json
{
  "experiment-name": {
    "match": {
      "sky": "blue",
      "clouds": "white"
    },
    "buckets": {
      "min": "0",
      "max": "50"
    }
  },
  "another-experiment-name": {
    "match": {
      "lang": "eng",
      "climber": "trad-only-bro"
    },
    "buckets": {
      "min": "0",
      "max": "50"
    }
  }
}
```

The `match` key is a json object that contains keys that map to string values.
Each key/value pair is an **exact** condition requirement for that experiment.
All key/value pairs **must** be satisfied for the experiment to be considered a match.

The `buckets` key is a json object that contains two keys, `min` and `max`.
`min` and `max` should be strings containing integer values, `0 <= x <= 100`
(switchboard is currently configured to have 100 buckets).
