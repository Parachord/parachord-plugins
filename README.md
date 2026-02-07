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

### AI Assistants
| Plugin | Description | Auth Required |
|--------|-------------|---------------|
| [ChatGPT](chatgpt.axe) | Generate playlists and chat using OpenAI | Yes |
| [Claude](claude.axe) | Anthropic's Claude AI assistant | Yes |
| [Gemini](gemini.axe) | Generate playlists and chat using Google Gemini | Yes |
| [Ollama](ollama.axe) | Run AI locally with Ollama - free, private, works offline | No |

Additional plugins (scrobblers, metadata providers, and more music sources) are available through the Parachord plugin marketplace. See [manifest.json](manifest.json) for the full catalog.

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
