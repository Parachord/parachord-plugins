# Parachord Plugins

Community plugins for [Parachord](https://github.com/Parachord/parachord) - the universal music player.

## What are Plugins?

Plugins extend Parachord's capabilities by adding support for music services, recommendations, scrobbling, concerts, AI features, and more. They're written as `.axe` files - JSON manifests with embedded JavaScript implementations.

New to writing one? Start with **[AUTHORING.md](AUTHORING.md)**, a walkthrough from an empty file to a working resolver.

## Available Plugins

### Music Sources (Resolvers)
| Plugin | Description | Auth Required |
|--------|-------------|---------------|
| [Amazon Music](amazonmusic.axe) | Play Amazon Music in Parachord's own player, with album search, artist photos, playlist import and your personal recommendations. Playing needs Music Unlimited; with Prime it's used for album art, artist photos, links and recommendations | Yes |
| [Apple Music](applemusic.axe) | Stream via MusicKit with search, URL lookup, album/playlist support and your personal recommendations | Yes |
| [Bandcamp](bandcamp.axe) | Find, play and purchase music on Bandcamp, through Bandcamp's own player | No |
| [Local Files](localfiles.axe) | Play local music files | No |
| [SoundCloud](soundcloud.axe) | Search and stream music from SoundCloud | Yes |
| [Spotify](spotify.axe) | Spotify integration via Connect API. Needs Spotify Premium on the account that owns your Spotify developer app | Yes |
| [Subsonic](subsonic.axe) | Stream from your own Subsonic-compatible server (Navidrome, Airsonic, Gonic, ...) | Yes |
| [YouTube](youtube.axe) | Play music from YouTube | No |

### Listening History and Recommendations
| Plugin | Description | Auth Required |
|--------|-------------|---------------|
| [Last.fm](lastfm.axe) | Scrobbling, recommendations from your Last.fm history, and listeners with similar taste | Yes |
| [ListenBrainz](listenbrainz.axe) | Scrobbling, recommendations and your weekly playlists, and listeners with similar taste | Yes |
| [Libre.fm](librefm.axe) | Scrobble to Libre.fm | Yes |

### Metadata
| Plugin | Description | Auth Required |
|--------|-------------|---------------|
| [Discogs](discogs.axe) | Artist biographies and images | No |
| [Wikipedia](wikipedia.axe) | Artist biographies and images | No |

### Concerts
| Plugin | Description | Auth Required |
|--------|-------------|---------------|
| [Bandsintown](bandsintown.axe) | Upcoming concerts for artists you love | Yes |
| [SeatGeek](seatgeek.axe) | Concerts and live events with ticket info | Yes |
| [Songkick](songkick.axe) | Live music events near you | Yes |
| [Ticketmaster](ticketmaster.axe) | Concerts and live events via the Ticketmaster Discovery API | Yes |

### AI Assistants
| Plugin | Description | Auth Required |
|--------|-------------|---------------|
| [ChatGPT](chatgpt.axe) | Generate playlists and chat using OpenAI | Yes |
| [Claude](claude.axe) | Anthropic's Claude AI assistant | Yes |
| [Custom LLM](customllm.axe) | Bring your own OpenAI- or Anthropic-compatible endpoint (LM Studio, vLLM, llama.cpp, OpenRouter, ...) | No |
| [Gemini](gemini.axe) | Generate playlists and chat using Google Gemini | Yes |
| [Ollama](ollama.axe) | Run AI locally with Ollama - free, private, works offline | No |

### Community
| Plugin | Description | Auth Required |
|--------|-------------|---------------|
| [Achordion](achordion.axe) | Shares verified track matches (MusicBrainz IDs and streaming links) so other listeners' resolvers match faster | No |

See [manifest.json](manifest.json) for the full plugin marketplace catalog.

## Creating a Plugin

Plugins are `.axe` files with the following structure. [AUTHORING.md](AUTHORING.md) walks through building one; the [plugin authoring guide](https://github.com/Parachord/parachord/blob/main/docs/architecture/plugins.md#authoring-a-plugin-axe-format) in the main repo is the full reference for every field and contract.

```json
{
  "manifest": {
    "id": "my-plugin",
    "name": "My Plugin",
    "version": "1.0.0",
    "author": "Your Name",
    "description": "What this plugin does",
    "icon": "🎵",
    "color": "#ff6b6b",
    "homepage": "https://example.com",
    "email": "you@example.com",
    "minAppVersion": { "desktop": "0.9.12" }
  },

  "capabilities": {
    "resolve": true,
    "search": true,
    "stream": false,
    "urlLookup": false
  },

  "urlPatterns": [
    "*.example.com/track/*"
  ],

  "settings": {
    "requiresAuth": false,
    "authType": "none",
    "configurable": {}
  },

  "implementation": {
    "search": "async function(query, config) { ... }",
    "resolve": "async function(artist, track, album, config) { ... }",
    "play": "async function(track, config) { ... }",
    "init": "async function(config) { ... }",
    "cleanup": "async function() { ... }"
  }
}
```

- **`manifest.id`** may contain only letters, digits, dots, dashes and underscores. **`version`** is semver; bump it on every change, or apps that already have your plugin never download the new copy.
- **`manifest.type`** is absent (or `"resolver"`) for a plugin that finds and plays tracks, and `"meta-service"` for everything else: recommendations, scrobbling, metadata, concerts, AI.
- **`manifest.minAppVersion`** (optional, per platform: `desktop`, `android`, `ios`) is the oldest app your plugin runs on. Apps below it keep the copy they have.

### Capabilities

Capability values are always `true` or `false`; the mobile apps read them as booleans and drop a plugin whose capabilities don't parse that way.

**Music sources (resolvers):**

- **resolve**: Can find a playable source for a given track
- **search**: Can search for tracks
- **stream**: Plays audio itself
- **urlLookup**: Can read tracks, albums or playlists from the service's links (with `urlPatterns`)
- **albumSearch**: Can find an album by name (`searchAlbums`; the album-page catalog fallback)
- **artistImage**: Can find a photo of an artist by name (`searchArtistImage`)
- **html5Stream**: The host plays the URL the plugin's `getStreamUrl` returns
- **widevine**: The host plays the service's DRM-protected stream
- **embedPlayer**: The host plays the track in the service's own embeddable player (SoundCloud's widget), through a host adapter named in a top-level `"embedPlayer": { "adapter": "<name>" }` block
- **resolvePicks**: The plugin's own `resolve` picks better than a generic pick among its search results, so playback calls `resolve`
- **purchase**: Shows a Buy button in the player
- **recommendations**: Feeds the Recommendations page (`getRecommendations`)

**Meta-services:**

- **recommendations**: Feeds the Recommendations page (`getRecommendations`, optionally `expandRecommendationSection`)
- **similarUsers**: Suggests listeners with similar taste (`getSimilarUsers`)
- **scrobbling**, **listeningHistory**: Scrobbling and listening data
- **metadata**, **biography**, **artistImages**: Artist information
- **concerts**: Upcoming shows (`searchArtist`, `searchArtistEvents`)
- **generate**: One-shot AI playlists
- **chat**: Conversational AI (the AI DJ)
- **playbackTelemetry**: Reports verified plays (Achordion)

`html5Stream`, `widevine` and `embedPlayer` hand playback to the app, which must support them. A meta-service's code loads only when it declares a capability that uses it (`generate`, `chat`, `concerts`, `playbackTelemetry`, `recommendations` or `similarUsers`) along with the matching function, or `urlLookup` with `urlPatterns`. A plugin that needs a newer app than some users have declares `manifest.minAppVersion`, and until most people have updated, it should still check that the host feature exists and fall back. The full list and each contract are in the [plugin authoring guide](https://github.com/Parachord/parachord/blob/main/docs/architecture/plugins.md#authoring-a-plugin-axe-format).

### Implementation Functions

Each function runs on its own: there's no shared scope between them, so a helper two functions need is defined inside each. Inside a function, `this` is the plugin, so `resolve` can call `this.search(query, config)`. Plugins run with the app's own privileges, so users should install only plugins they trust.

#### Resolver Functions
| Function | Purpose | Parameters |
|----------|---------|------------|
| `init` | Initialize the plugin | `config` |
| `cleanup` | Clean up resources | none |
| `search` | Search for tracks: an array of tracks, `[]` when nothing matches | `query`, `config` |
| `resolve` | Find the best playable match for one track: a track, or `null` when there's none | `artist`, `track`, `album`, `config` |
| `play` | Play a track: truthy on success, falsy to try the next source, or throw `plan-ineligible` when the account's plan can't play it (see below) | `track`, `config` |
| `lookupUrl` | Get track info from URL | `url`, `config` |
| `lookupAlbum` | Get album tracks from URL | `url`, `config` |
| `lookupPlaylist` | Get playlist tracks from URL | `url`, `config` |
| `searchAlbums` | Find an album by artist and title (with `albumSearch`) | `query`, `config` |
| `searchArtistImage` | Find a photo of an artist (with `artistImage`) | `artistName`, `config` |
| `getStreamUrl` | The URL the host plays (with `html5Stream`); build it from the saved `config`, never from fields stored on the track | `track`, `config` |
| `isConfigured` | Whether the saved settings are complete, so the Plug-Ins page can flag a plugin that still needs setting up | `config` |

#### Recommendation Functions
| Function | Purpose | Parameters |
|----------|---------|------------|
| `getRecommendations` | `{ sections: [{ id?, title?, description?, nextCursor?, items }] }`. Each item is `{ type: 'artist' \| 'album' \| 'song' \| 'playlist', name, artist?, id?, imageUrl?, url?, reason?, duration?, mbid?, isrc?, releaseDate? }` (`releaseDate` on albums; the page's "Coming" badge uses it) | `config` |
| `expandRecommendationSection` | The next page of a titled section: `{ items, nextCursor }` | `sectionId`, `afterCursor`, `config` |
| `getSimilarUsers` | `[{ username, similarity, avatarUrl?, topArtists? }]`, most similar first (`similarity` 0 to 1) | `config` |

A service that isn't configured answers empty without a request. A miss is `{ sections: [] }` or `[]`; a failure throws (see the rules below).

#### Plans that can't play on demand

Some services have a tier that can search, browse and sync but can't play a chosen track on demand, such as Amazon Prime (vs Music Unlimited). Parachord 0.9.12 and later keep such a plugin enabled for what it still does (search, album art, artist photos, links, playlists) and leave it out of playback: no source icons, no playback priority number, never picked as a source, and Settings says why.

- **When the service refuses a play because of the plan**, `play()` throws an error with `reason: 'plan-ineligible'` and, if known, `planKey` (the plan as the service names it, e.g. `'free'`), instead of returning `false`. A plain `false` reads as a missing device or a bad match. Throw it only when `window.__parachordNotePlanRefusal` exists, return `false` otherwise (older apps and the mobile clients), and declare `"minAppVersion": { "desktop": "0.9.12" }`. Treat only a refusal the service gives for the plan specifically as `plan-ineligible`, not any 403. No plugin here does this yet: Amazon Music's refusal comes from the app's own player.
- **If the service also refuses searches on that plan**, report it as not-ready (`err.notReady = true`, `err.code = 'plan-ineligible'`) rather than an empty result, so nothing is cached as a miss and an upgrade searches again.

The details are in the [plugin authoring guide](https://github.com/Parachord/parachord/blob/main/docs/architecture/plugins.md#plans-that-cant-play-on-demand-plan-ineligible).

#### AI Functions
| Function | Purpose | Parameters |
|----------|---------|------------|
| `chat` | Conversational AI interaction | `messages`, `tools`, `config` |
| `generate` | Generate playlists from prompts | `prompt`, `context`, `config` |
| `listModels` | The models a settings dropdown offers | `config` |

### Rules Every Resolver Follows

- **A miss is empty; a failure throws.** An empty `search` or a `null` `resolve` tells Parachord "this service doesn't have it", and the app remembers that for the track (a week, and 30 days in saved playlists). So when the service refused the request (401, 403, 429), failed (5xx), timed out or the network dropped, throw an error instead; nothing is remembered, and the app asks again later. Don't wrap a whole function in a `try` whose `catch` returns `[]`. For "not signed in" or "not set up", throw an error with `notReady = true` and a short `code`.
- **`resolve` has 8 seconds.** After that the app moves on without it and asks again later. Keep requests few and sequential.
- **Match the way the app matches.** The app accepts a result only when both artist and title match. If your `resolve` filters or ranks its own results, prefer the result whose artist and title both match the request, rather than returning the service's top hit.
- **Pass `config` through.** When `resolve` calls `this.search`, hand it the `config` it was given: the app uses it to put a play click ahead of background work.
- **Respect the service.** Check its minimum query length, stay within its rate limits, never hardcode API keys (use `settings.configurable`), and follow the copy-protection rule in the guidelines below.

### Resolver Result Shape

`search` and `resolve` return result objects that get spread into `track.sources[resolverId]` via `{ ...result, confidence }`. The shape is conventional rather than typed — fields a resolver populates show up as `track.sources[resolverId].<field>`. Required fields are obvious from the resolver implementations (`id`, `title`, `artist`, `album`, `duration`). The interesting set is the **optional cross-cutting metadata** that downstream consumers walk for, regardless of which resolver populated it:

| Field | Type | When to populate |
|-------|------|------------------|
| `isrc` | `string` | Whenever the upstream API exposes a recording ISRC. Normalize to upper-case + trimmed, format `[A-Z]{2}[A-Z0-9]{3}\d{7}`. Validate before storing — drop malformed values rather than emitting them. Currently captured by: Spotify (`external_ids.isrc` on `/v1/search`, `/v1/tracks/{id}`, `/v1/playlists/{id}/tracks`), Apple Music native MusicKit (`song.isrc` from catalog), local files (ID3 TSRC / Vorbis ISRC / MP4 atom via music-metadata's `common.isrc`). Bandcamp / SoundCloud / YouTube / iTunes Search don't expose ISRC; leave undefined. |

#### Why cross-cutting metadata goes on the source record

Resolver-specific identifiers (`spotifyId`, `appleMusicId`, `bandcampUrl`, `youtubeId`) stay on the source record for the resolver that owns them — they only make sense in that context. Cross-cutting metadata like ISRC identifies the underlying *recording*, not the service's view of it, so it belongs alongside the per-source fields and downstream consumers can walk `track.sources[*].isrc` to accept the first valid value from any resolver that captured it. The same principle generalizes to other recording-keyed metadata that might land later (BPM, key, recording-MBID, etc.).

Consumers in the Parachord desktop client use `window.pickTrackIsrc(track)` (defined in `app.js`) which checks top-level `track.isrc` first, then walks sources, skipping `noMatch` entries. Current consumers: the ISRC → recording-MBID fallback in `resolveMbidForLove` and `enrichTrackWithMbid`, and the Achordion `track-links/submit` ISRC-only fallback. New consumers should reuse `pickTrackIsrc` rather than reading individual fields directly.

## Contributing

1. Fork this repository
2. Create your `.axe` plugin file in the repository root ([AUTHORING.md](AUTHORING.md) shows how)
3. Test it locally in Parachord (Settings → Plug-Ins → Add from file)
4. Submit a pull request

### Guidelines

- Follow the existing plugin structure
- Include clear error handling
- Don't hardcode API keys - use the config system
- Test with various edge cases
- Update this README with your plugin
- Respect each service's copy protection. A plugin must not decrypt, record, download or save protected streams, extract decryption keys, or unlock tiers or content the user's subscription doesn't include. Protected audio has to play with the user's own account, through the service's own player, its official API, or the same endpoints its own web player uses, with the stream and its license coming from the service. Where the service counts plays, the plugin has to report them to it the way the service's own player does. A plugin that tries to get around a service's protections won't be accepted, and is removed from the marketplace if one slips through. (Parachord's own stance is in the [main README](https://github.com/Parachord/parachord#subscriptions-and-copy-protection).)

## License

MIT License - see [LICENSE](LICENSE) for details.

Plugins are community-contributed and may have their own licenses specified in their manifests.
