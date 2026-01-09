# Masivotech Claude Code Plugins

A curated collection of Claude Code plugins by [masivotech](https://masivo.tech).

## Installation

Add this marketplace to Claude Code:

```bash
/plugin marketplace add masivotech/masivotech-claude-code
```

## Available Plugins

### intellij-plugin-dev

Comprehensive skills, commands, and agent for JetBrains/IntelliJ Platform Plugin SDK development.

**Install:**
```bash
/plugin install intellij-plugin-dev@masivotech-plugins
```

**Features:**
- **Skill**: Automatically triggered when working on IntelliJ plugin development
- **Commands**:
  - `/intellij:new-action` - Scaffold a new AnAction
  - `/intellij:new-service` - Scaffold a new Service
  - `/intellij:new-tool-window` - Scaffold a new Tool Window
  - `/intellij:verify` - Run Plugin Verifier
  - `/intellij:compat` - Check IDE compatibility
- **Agent**: `plugin-architect` for plugin architecture design

**Supported IDEs:**
- IntelliJ IDEA (Community & Ultimate)
- Android Studio
- PyCharm, WebStorm, GoLand, PhpStorm
- RubyMine, CLion, Rider

## Requirements

- Claude Code v2.0.12 or higher

## License

MIT License

## Author

[masivotech](https://github.com/masivotech)
