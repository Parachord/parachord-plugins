# Building a Parachord Plugin

This walkthrough builds a small resolver, a plugin that finds tracks on a music service and lets Parachord play them, from an empty folder to an `.axe` file you can install and test. The service is made up ("Example Music" at `api.example.com`), so you can follow the shape without an account. Swap in your service's real endpoints as you go.

For every field and contract in detail, see the [plugin authoring guide](https://github.com/Parachord/parachord/blob/main/docs/architecture/plugins.md#authoring-a-plugin-axe-format) in the main repo. The [README](README.md) lists every capability and function.

## What a plugin is

An `.axe` file is JSON with a few top-level keys:

- `manifest`: who the plugin is (id, name, version, ...)
- `capabilities`: what it can do, as `true`/`false` flags
- `settings`: what the user fills in (an API key, a server URL, ...)
- `urlPatterns` (optional): links your plugin understands
- `implementation`: the code, as a map of function name to the function's source text

Writing JavaScript inside JSON strings by hand is painful, so this walkthrough keeps the code in a normal `.js` file and turns it into the `.axe` with a tiny script.

## 1. Lay out the files

```
examplemusic/
  manifest.json       everything except the code
  implementation.js   the functions
build-axe.js          turns the two into examplemusic.axe
```

## 2. The manifest, capabilities and settings

`examplemusic/manifest.json`:

```json
{
  "manifest": {
    "id": "examplemusic",
    "name": "Example Music",
    "version": "1.0.0",
    "author": "Your Name",
    "description": "Find and play tracks from Example Music.",
    "icon": "🎧",
    "color": "#3b82f6",
    "homepage": "https://example.com"
  },
  "capabilities": {
    "resolve": true,
    "search": true,
    "stream": true,
    "html5Stream": true
  },
  "settings": {
    "requiresAuth": true,
    "authType": "apikey",
    "configurable": {
      "apiKey": {
        "type": "password",
        "label": "API key",
        "description": "From your Example Music account page."
      }
    }
  }
}
```

- `id` may use only letters, digits, dots, dashes and underscores, and it can't change later.
- `html5Stream` means Parachord plays a plain audio URL your plugin gives it (from `getStreamUrl`). Services with DRM or their own embeddable player use other capabilities; see the guide.
- Each `configurable` field becomes an input in Settings → Plug-Ins, and its value reaches every function as `config.<key>` (here `config.apiKey`). Field types in use are `text`, `password`, `select` and `dynamic-select`.

## 3. The functions

`examplemusic/implementation.js`:

```js
module.exports = {
  // Called when the plugin loads.
  init: async function (config) {},

  // Lets the Plug-Ins page flag the plugin until it's set up.
  isConfigured: function (config) {
    return !!(config && config.apiKey);
  },

  // An array of tracks. [] means "nothing matches".
  search: async function (query, config) {
    if (!config || !config.apiKey) return []; // not set up: a quiet miss, no request
    var q = String(query || '').trim();
    if (q.length < 2) return [];

    var res = await fetch(
      'https://api.example.com/v1/search?q=' + encodeURIComponent(q) +
      '&key=' + encodeURIComponent(config.apiKey)
    );
    // A refusal or an outage is NOT "nothing matches": throw, so Parachord
    // doesn't remember these tracks as missing from Example Music.
    if (!res.ok) throw new Error('Example Music search failed: ' + res.status);

    var data = await res.json();
    return (data.tracks || []).map(function (t) {
      return {
        id: 'examplemusic-' + t.id,
        title: t.title,
        artist: t.artist,
        album: t.album || '',
        duration: Math.round((t.durationMs || 0) / 1000), // seconds
        albumArt: t.coverUrl || null,
        sources: ['examplemusic'],
        exampleMusicId: String(t.id) // your service's own id, used to play it
      };
    });
  },

  // The best match for one track, or null when the service doesn't have it.
  resolve: async function (artist, track, album, config) {
    var norm = function (s) {
      return String(s || '').toLowerCase().replace(/[^a-z0-9]+/g, '');
    };
    // Pass config through: Parachord uses it to put a play click ahead of
    // background lookups.
    var results = await this.search(artist + ' ' + track, config);
    var wantArtist = norm(artist);
    var wantTitle = norm(track);
    for (var i = 0; i < results.length; i++) {
      if (norm(results[i].artist) === wantArtist && norm(results[i].title) === wantTitle) {
        return results[i];
      }
    }
    return null;
  },

  // The URL Parachord plays. Build it from the saved settings plus the
  // track's own id; never store credentials on the track.
  getStreamUrl: async function (track, config) {
    return 'https://api.example.com/v1/stream/' + encodeURIComponent(track.exampleMusicId) +
      '?key=' + encodeURIComponent(config.apiKey);
  },

  // With html5Stream, Parachord plays the audio itself, so play() has
  // nothing to do.
  play: async function (track, config) {
    return false;
  },

  cleanup: async function () {}
};
```

Things to know about these functions:

- **Write each one as `name: async function (...) { ... }`.** Method shorthand (`async search(query) {}`) doesn't survive being turned into text.
- **Each function runs on its own.** There's no shared scope, so a helper two functions need (like `norm`) is defined inside each. Inside a function, `this` is the plugin, so `resolve` can call `this.search(...)`.
- **A miss is empty; a failure throws.** Parachord remembers an empty `search` or a `null` `resolve` as "not on this service" for a week (30 days in saved playlists). Only answer empty when the service really had nothing. If you can't tell (a 401, 403, 429, 5xx, a timeout, no network), throw. For "not signed in" or "not set up", throw an error with `notReady = true` and a short `code`.
- **`resolve` has 8 seconds.** After that Parachord moves on without it and asks again later.
- **Match on artist and title.** Parachord accepts a result only when both match. Returning the service's top hit for every search makes wrong versions show up as matches.

## 4. Build the .axe

`build-axe.js`:

```js
// Usage: node build-axe.js examplemusic
const fs = require('fs');
const id = process.argv[2];
const base = JSON.parse(fs.readFileSync(`./${id}/manifest.json`, 'utf8'));
const impl = require(`./${id}/implementation.js`);

const implementation = {};
for (const [name, fn] of Object.entries(impl)) implementation[name] = String(fn);

fs.writeFileSync(`${id}.axe`, JSON.stringify({ ...base, implementation }, null, 2) + '\n');
console.log(`Wrote ${id}.axe`);
```

```bash
node build-axe.js examplemusic
```

## 5. Install and test it

1. In Parachord, open **Settings → Plug-Ins** and click **Add from file**. Pick `examplemusic.axe`. It loads straight away, no restart.
2. Open the plugin's settings and fill in your API key.
3. Search for a song the service has. Your results should show the Example Music icon, and playing one should play your stream.
4. Open an album or playlist page: tracks the service has should get your source icon as they resolve.

To debug, open the developer tools (Help → Toggle Developer Tools) and watch the console. Log with a prefix (`console.log('[ExampleMusic] ...')`) so your lines are easy to find.

After a change, rebuild and add the file again: it replaces the copy you installed and loads straight away. Bump `version` before you share a change: apps that already have your plugin only download a newer version.

## 6. Going further

- **Links:** add `urlPatterns` (for example `"example.com/track/*"`) plus `urlLookup: true` and `lookupUrl(url, config)` (and `lookupAlbum` / `lookupPlaylist`), so pasted links and imports work. Whether a link counts as a track, album or playlist is decided inside the app for each known service, so a new service's links may need a small change in the main repo too.
- **Album pages and artist photos:** `albumSearch` with `searchAlbums(query, config)`, and `artistImage` with `searchArtistImage(artistName, config)`.
- **Recommendations:** `recommendations` with `getRecommendations(config)` feeds the Recommendations page; `similarUsers` with `getSimilarUsers(config)` suggests listeners. The item shapes are in the [README](README.md#recommendation-functions).
- **Newer app features:** if your plugin needs something only newer versions of Parachord have, declare `"minAppVersion": { "desktop": "x.y.z" }` in the manifest, and still check that the feature exists before using it.
- **Mobile:** the Android and iOS apps run plugins too, in their own JavaScript engines. Avoid desktop-only APIs (anything under `window.electron`), and guard modern regex features with a fallback. If your plugin can't work on mobile, add `"mobile": false` to its capabilities so the apps don't offer it.

## 7. Before you submit

- [ ] `id` is final, and `version` is bumped for this change
- [ ] No API keys or secrets in the code; everything comes from `settings.configurable`
- [ ] An unconfigured plugin answers empty without making a request
- [ ] A miss is `[]` / `null`; a refusal, error or timeout throws
- [ ] `resolve` returns a result only when artist and title both match
- [ ] `resolve` passes `config` to `this.search`, and finishes well within 8 seconds
- [ ] The service's rate limits and minimum query length are respected
- [ ] The plugin follows the copy-protection rule in the [README guidelines](README.md#guidelines): the user's own account, the service's own player or API, the stream and license from the service, plays reported as the service's own player does
- [ ] You tested it with Add from file, and added it to the README's plugin list

Then open a pull request with your `.axe` file in the repository root. I'm the only maintainer right now, so reviews can take a little while; questions in the PR are welcome.
