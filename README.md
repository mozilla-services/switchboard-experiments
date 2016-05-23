# switchboard-experiments
![travis-ci](https://travis-ci.org/mozilla-services/switchboard-experiments.svg?branch=master)

This repository contains the JSON file that configures the [switchboard](https://github.com/mozilla-services/switchboard-server) expermients that are active on [Firefox for Android](https://developer.mozilla.org/en-US/docs/Simple_Firefox_for_Android_build).

Any changes to this file require the approval of a Firefox for Android peer, such as [@leibovic](https://github.com/leibovic), [@liuche](https://github.com/liuche), or [@mfinkle](https://github.com/mfinkle). Additionally, changes that affect release branches (i.e. Aurora/Beta/Release), must have approval from our product and release management teams.

## Deployments

* `switchboard.services.mozilla.com` is the production endpoint. It pulls experiments from the `master` branch. There is also a legacy `switchboard-server.dev.mozaws.net` endpoint, which also points the the production deployment.
* `switchboard.stage.mozaws.net` is the staging endpoint. It pulls experiments from the `stage` branch.

## Experiment Defintions

Experiment names are defined in the client in [Experiments.java](http://hg.mozilla.org/mozilla-central/file/tip/mobile/android/base/java/org/mozilla/gecko/util/Experiments.java) and in the `Experiments` object in [browser.js](http://hg.mozilla.org/mozilla-central/file/tip/mobile/android/chrome/content/browser.js);

UI experiments:
* `bookmark-history-menu`: Display "History" and "Bookmarks" menu items in 3-dot menu.
* `malware-download-protection`: Enable malware download protection.
* `offline-cache`: Try to load pages from disk cache when network is offline.
* `search-term`: Show search mode (instead of home panels) when tapping on urlbar if there is a search term in the urlbar.
* `whatsnew-notification`: Show a "What's new" notification when the browser updates. Tapping on this notification will open a new tab with a SUMO article about what is new in Firefox.
* `content-notifications-12hrs`: Enable content notifications and check for updates every 12 hours at random times based on app start.
* `content-notifications-8am`: Enable content notifications and check for updates every day at 8 am.
* `content-notifications-5pm`: Enable content notifications and check for updates every day at 5 pm.
* `promote-add-to-homescreen`: Show prompt to add the current website to the home screen if this website is visited frequently ([bug 1232706](https://bugzilla.mozilla.org/show_bug.cgi?id=1232706))
* `triple-readerview-bookmark-prompt`: Show prompt to bookmark the current page if it has been reader-viewed 3 times ([bug 1247689](https://bugzilla.mozilla.org/show_bug.cgi?id=1247689))
* `urlbar-show-origin-only`: Only show origin in URL bar instead of full URL ([bug 1236431](https://bugzilla.mozilla.org/show_bug.cgi?id=1236431))
* `urlbar-show-ev-cert-owner`: Show name of organization (EV cert) instead of full URL in URL bar ([bug 1249594](https://bugzilla.mozilla.org/show_bug.cgi?id=1249594))
* `download-content-catalog-sync`: Synchronize catalog of downloadable content from Kinto (Staged rollout - [bug 1271352](https://bugzilla.mozilla.org/show_bug.cgi?id=1271352))

Onboarding experiments are unique because we use local logic to determine whether a client is in an experiment. We do this because we must know if the experiment is active at startup, and we cannot wait to contact the Switchboard server. Given this fact, changes to `experiments.json` will not affect onboarding experiments. Those experiments are maintained in the client codebase.

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
Each key/value pair is a regular expression match requirement for that experiment.
Regular expressions are matched by the node backend, and follow [this](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/RegExp) format. Note: this means that if you want an **exact** string match, as opposed to a RegExp that will match strings that **contain** your specified string, you must use string delimiters (^ and $ for start and end of string respectively).
All key/value pairs **must** be satisfied for the experiment to be considered a match.

Supported keys are specified by the client, as parameters on the [server request](http://hg.mozilla.org/mozilla-central/file/494289c72ba3/mobile/android/thirdparty/com/keepsafe/switchboard/SwitchBoard.java#l226). Here is a list of keys that are currently supported in Firefox for Android:
* `appId`: The Android app ID (e.g. `org.mozilla.fennec`, `org.mozilla.firefox_beta`, `org.mozilla.firefox`)
* `version`: The Firefox app version number (e.g. `47.0a1'`, `46.0`)
* `lang`: Language, pulled from the default locale (e.g. `eng`)
* `country`: Country, pulled from the default locale (e.g. `USA`)
* `device`: Android device name
* `manufacturer`: Android device manufacturer

### `buckets` Key

The `buckets` key is a JSON object that contains two keys, `min` and `max`.
`min` and `max` should be strings containing integer values, `0 <= min <= max <= 100`. The bounds are [min, max).

The experiment "experiment-name" below will trigger for buckets 0-49:

```
"experiment-name": {
  "match": {},
  "buckets": {
    "min": "0",
    "max": "50"
  }
}
```

Switchboard is currently configured to have 100 buckets (0-99), but the max value is a non-inclusive upper bound, meaning that unless a value of 100 or higher is used as the max, bucket 99 will not be included. There is no check for out of bounds values, so "disabled" experiments can have min/max buckets keys that are either out of bounds or have min == max.

## Making Client Changes

To use Switchboard to expose a new feature to a portion of Firefox users, use a code snippet like this in the client:

```java
if (SwitchBoard.isInExperiment(this, Experiments.YOUR_EXPERIMENT_NAME)) {
  // Do something interesting.
}
```
You should define your experiment in [Experiments.java](http://hg.mozilla.org/mozilla-central/file/tip/mobile/android/base/java/org/mozilla/gecko/util/Experiments.java), and then add the same experiment name definition to `experiments.json`. All new experiment names must be documented in this repo.

To force enable a feature for testing on the client, you can use the [switchboard-experiments add-on](https://addons.mozilla.org/en-US/android/addon/switchboard-experiments/). Note: Support for this has only landed in Firefox 49, but may be uplifted to 47 or 48.

## Making Config Changes

You can use the `stage` branch to test your config changes. Once you push a change to `stage`, you can start Fennec with an intent extra to specify a custom Switchboard server host:

`adb shell am start --es "switchboard-host" "switchboard.stage.mozaws.net" <package-name>`

Support for this intent extra is currently on Nightly only, but once this is in all release channels, we should use this to test config changes against all versions of the client.

