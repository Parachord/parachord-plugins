# Parachord Plugins

Community plugins for [Parachord](https://github.com/Parachord/parachord) - the universal music player.

## What are Plugins?

Plugins extend Parachord's capabilities by adding support for music services, scrobbling, AI features, and more. They're written as `.axe` files - JSON manifests with embedded JavaScript implementations.

## Available Plugins

### Music Sources (Resolvers)
| Plugin | Description | Auth Required |
|--------|-------------|---------------|
| [Bandcamp](resolvers/bandcamp.axe) | Find and play music on Bandcamp | No |
| [YouTube](resolvers/youtube.axe) | Play music from YouTube | No |
| [Spotify](resolvers/spotify.axe) | Spotify integration | Yes |
| [Apple Music](resolvers/applemusic.axe) | Apple Music integration | Yes |
| [Local Files](resolvers/localfiles.axe) | Play local music files | No |

### Scrobblers
| Plugin | Description | Auth Required |
|--------|-------------|---------------|
| [Last.fm](resolvers/lastfm.axe) | Scrobble to Last.fm | Yes |
| [ListenBrainz](resolvers/listenbrainz.axe) | Scrobble to ListenBrainz | Yes |
| [Libre.fm](resolvers/librefm.axe) | Scrobble to Libre.fm | Yes |

### Metadata
| Plugin | Description | Auth Required |
|--------|-------------|---------------|
| [Discogs](resolvers/discogs.axe) | Album and artist metadata | No |
| [Wikipedia](resolvers/wikipedia.axe) | Artist biographies | No |

### AI Assistants
| Plugin | Description | Auth Required |
|--------|-------------|---------------|
| [ChatGPT](resolvers/chatgpt.axe) | OpenAI integration | Yes |
| [Gemini](resolvers/gemini.axe) | Google AI integration | Yes |

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

### Implementation Functions

| Function | Purpose | Parameters |
|----------|---------|------------|
| `init` | Initialize the plugin | `config` |
| `cleanup` | Clean up resources | none |
| `search` | Search for tracks | `query`, `config` |
| `resolve` | Find playable source | `artist`, `track`, `album`, `config` |
| `play` | Play a track | `track`, `config` |
| `lookupUrl` | Get track info from URL | `url`, `config` |
| `lookupAlbum` | Get album tracks from URL | `url`, `config` |

## Contributing

1. Fork this repository
2. Create your plugin in `resolvers/`
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
