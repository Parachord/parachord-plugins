# Parachord Plugins

Community plugins for [Parachord](https://github.com/Parachord/parachord) - the universal music player.

## What are Plugins?

Plugins extend Parachord's capabilities by adding support for music services, scrobbling, AI features, and more. They're written as `.axe` files - JSON manifests with embedded JavaScript implementations.

## Available Plugins

### Music Sources (Resolvers)
| Plugin | Description | Auth Required |
|--------|-------------|---------------|
| [Apple Music](applemusic.axe) | Stream via MusicKit with search, URL lookup, and album/playlist support | Yes |
| [Bandcamp](bandcamp.axe) | Find and purchase music on Bandcamp | No |
| [SoundCloud](soundcloud.axe) | Search and stream music from SoundCloud | Yes |
| [Spotify](spotify.axe) | Spotify integration via Connect API | Yes |
| [YouTube](youtube.axe) | Play music from YouTube | No |
| [Local Files](localfiles.axe) | Play local music files | No |

### Scrobblers
| Plugin | Description | Auth Required |
|--------|-------------|---------------|
| [Last.fm](lastfm.axe) | Scrobble to Last.fm | Yes |
| [ListenBrainz](listenbrainz.axe) | Scrobble to ListenBrainz | Yes |
| [Libre.fm](librefm.axe) | Scrobble to Libre.fm | Yes |

### Metadata
| Plugin | Description | Auth Required |
|--------|-------------|---------------|
| [Discogs](discogs.axe) | Album and artist metadata | No |
| [Wikipedia](wikipedia.axe) | Artist biographies | No |

### AI Assistants
| Plugin | Description | Auth Required |
|--------|-------------|---------------|
| [ChatGPT](chatgpt.axe) | Generate playlists and chat using OpenAI | Yes |
| [Claude](claude.axe) | Anthropic's Claude AI assistant | Yes |
| [Gemini](gemini.axe) | Generate playlists and chat using Google Gemini | Yes |
| [Ollama](ollama.axe) | Run AI locally with Ollama - free, private, works offline | No |

See [manifest.json](manifest.json) for the full plugin marketplace catalog.

## Creating a Plugin

Plugins are `.axe` files with the following structure:

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
    "email": "you@example.com"
  },

  "capabilities": {
    "resolve": true,
    "search": true,
    "stream": false,
    "browse": false,
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

### Capabilities

- **resolve**: Can find playable sources for a track
- **search**: Can search for tracks/albums/artists
- **stream**: Provides direct audio streams
- **browse**: Can browse catalogs/charts/playlists
- **urlLookup**: Can extract track info from URLs
- **chat**: Can handle conversational AI interactions

### Implementation Functions

#### Resolver Functions
| Function | Purpose | Parameters |
|----------|---------|------------|
| `init` | Initialize the plugin | `config` |
| `cleanup` | Clean up resources | none |
| `search` | Search for tracks | `query`, `config` |
| `resolve` | Find playable source | `artist`, `track`, `album`, `config` |
| `play` | Play a track | `track`, `config` |
| `lookupUrl` | Get track info from URL | `url`, `config` |
| `lookupAlbum` | Get album tracks from URL | `url`, `config` |
| `lookupPlaylist` | Get playlist tracks from URL | `url`, `config` |

#### AI Functions
| Function | Purpose | Parameters |
|----------|---------|------------|
| `chat` | Conversational AI interaction | `messages`, `tools`, `config` |
| `generate` | Generate playlists from prompts | `prompt`, `context`, `config` |

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
2. Create your `.axe` plugin file in the repository root
3. Test it locally in Parachord
4. Submit a pull request

### Guidelines

- Follow the existing plugin structure
- Include clear error handling
- Don't hardcode API keys - use the config system
- Test with various edge cases
- Update this README with your plugin

## License

MIT License - see [LICENSE](LICENSE) for details.

Plugins are community-contributed and may have their own licenses specified in their manifests.
