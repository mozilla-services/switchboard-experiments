# switchboard-experiments
![travis-ci](https://travis-ci.org/mozilla-services/switchboard-experiments.svg?branch=master)

This repository contains the JSON file that configures the [switchboard](https://github.com/mozilla-services/switchboard-server) expermients that are active on [Firefox for Android](https://developer.mozilla.org/en-US/docs/Simple_Firefox_for_Android_build).

Any changes to this file require the approval of a Firefox for Android peer, such as [@leibovic](https://github.com/leibovic), [@liuche](https://github.com/liuche), or [@mfinkle](https://github.com/mfinkle). Additionally, changes that affect release branches (i.e. Aurora/Beta/Release), must have approval from our product and release management teams.

## Experiment Defintions

Experiment names are defined in the client in [Experiments.java](http://hg.mozilla.org/mozilla-central/file/tip/mobile/android/base/java/org/mozilla/gecko/util/Experiments.java).

UI experiments:
* `bookmark-history-menu`: Display History and Bookmarks in 3-dot menu
* `search-term`: Show search mode (instead of home panels) when tapping on urlbar if there is a search term in the urlbar

Onboarding experiment #1 (released in Firefox 43):
* `onboarding-a`: Single Welcome screen
* `onboarding-b`: Welcome screen, Import screen

Onboarding experiment #2 (released in Firefox 46):
* `onboarding2-a`: Single (blue) Welcome screen
* `onboarding2-b`: 4 static feature slides
* `onboarding2-c`: 4 static + 1 clickable (Data saving) feature slides

Experiment names **should not** be reused. Becuase we have one config for all clients, we do not have a way to guarantee which version of an experiment is active. So instead, we use new experiment names.

## `experiments.json` Format

The format of `experiments.json` is a set of experiment name keys, each of which maps to an object with `match` and `buckets` keys. These keys determine if the experiment is active or not for a given client.

For example:

```json
{
  "experiment-name": {
    "match": {
      "appId": "org.mozilla.fennec"
    },
    "buckets": {
      "min": "0",
      "max": "50"
    }
  },
  "another-experiment-name": {
    "match": {
      "lang": "eng"
    },
    "buckets": {
      "min": "0",
      "max": "50"
    }
  }
}
```
### `match` Key

The `match` key is a JSON object that contains keys that map to string values.
Each key/value pair is an **exact** condition requirement for that experiment.
All key/value pairs **must** be satisfied for the experiment to be considered a match. Note: when creating experiments, this means that there will be a *lot* of duplication. Help me fix that!

Here is a list of keys that are currently supported:
* `appId`: The Android app ID (e.g. `org.mozilla.fennec`, `org.mozilla.firefox_beta`, `org.mozilla.firefox`)
* ...

### `buckets` Key

The `buckets` key is a JSON object that contains two keys, `min` and `max`.
`min` and `max` should be strings containing integer values, `0 <= x <= 100`
(switchboard is currently configured to have 100 buckets).

## Making Client Changes

To use Switchboard to expose a new feature to a portion of Firefox users, use a code snippet like this in the client:

```java
if (SwitchBoard.isInExperiment(this, Experiments.YOUR_EXPERIMENT_NAME)) {
  // Do something interesting.
}
```

You should define your experiment in [Experiments.java](http://hg.mozilla.org/mozilla-central/file/tip/mobile/android/base/java/org/mozilla/gecko/util/Experiments.java), and then add the same experiment name definition to `experiments.json`. All new experiment names must be documented in this repo.

## Testing Changes Locally

To test your config changes in a local Firefox build, follow these steps. 

1. `git clone git@github.com:mozilla-services/switchboard-server.git` (or your own fork)
2. `cd switchboard-server`
3. `npm install`
4. `EXPERIMENTS_FILE=path/to/your/experiments.json node index.js`
5. Get a URL for your local server (margaret used [localtunnel](https://localtunnel.me/))
6. Update the Switchboard default server URLs in [BrowserApp.java](http://hg.mozilla.org/mozilla-central/file/c0ba5835ca48/mobile/android/base/java/org/mozilla/gecko/BrowserApp.java#l587) to match your local server URL
7. Rebuild and run Fennec

