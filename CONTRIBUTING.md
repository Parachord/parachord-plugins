# Contributing to Parachord Plugins

Thanks for your interest in contributing! This guide will help you create and submit plugins.

## Getting Started

1. **Fork** this repository
2. **Clone** your fork locally
3. **Create a branch** for your plugin: `git checkout -b add-my-plugin`

## Creating a Plugin

### 1. Choose a Plugin Type

- **Resolver**: Adds a music source (Bandcamp, SoundCloud, etc.)
- **Scrobbler**: Sends listening history to a service
- **Metadata**: Provides artist/album information
- **AI**: Integrates AI assistants

### 2. Create Your .axe File

Create a new file in `resolvers/` named `yourplugin.axe`.

Use an existing plugin as a template - [bandcamp.axe](resolvers/bandcamp.axe) is a good example for resolvers.

### 3. Required Fields

Your manifest must include:
- `id`: Unique lowercase identifier (e.g., `"my-plugin"`)
- `name`: Display name
- `version`: Semver version (e.g., `"1.0.0"`)
- `author`: Your name or handle
- `description`: Brief description of what it does

### 4. Implementation Tips

```javascript
// Always handle errors gracefully
async function search(query, config) {
  try {
    const response = await fetch(url);
    if (!response.ok) {
      console.error('Search failed:', response.status);
      return [];
    }
    // ... process results
  } catch (error) {
    console.error('Search error:', error);
    return [];
  }
}
```

- Return empty arrays `[]` on failure, not errors
- Log errors with `console.error()` for debugging
- Use `config` for user settings, never hardcode credentials
- Handle rate limiting gracefully

## Testing

1. Copy your `.axe` file to Parachord's resolvers directory
2. Restart Parachord
3. Test all implemented functions
4. Check the console for errors

## Submitting

1. **Commit** your changes: `git commit -m "Add MyPlugin resolver"`
2. **Push** to your fork: `git push origin add-my-plugin`
3. **Open a Pull Request** with:
   - Description of what your plugin does
   - Any API keys or accounts needed for testing
   - Screenshots if applicable

## Code Review

We'll review your PR for:
- Correct manifest structure
- Error handling
- No hardcoded credentials
- Clean, readable code
- Proper licensing

## Questions?

Open an issue if you need help or have questions about the plugin API.
