# IntelliJ Plugin Development - Claude Code Plugin

A comprehensive Claude Code plugin for JetBrains/IntelliJ Platform Plugin SDK development. Includes skills, commands, and a specialized agent to help you build plugins for IntelliJ IDEA, Android Studio, and other JetBrains IDEs.

## Features

### Skill: IntelliJ Plugin Development Guide

Automatically triggered when Claude detects you're working on IntelliJ plugin development. Provides comprehensive guidance on:

- Plugin project structure and configuration
- Extension points (Actions, Services, Tool Windows, etc.)
- Gradle IntelliJ Plugin 2.x configuration
- PSI (Program Structure Interface) patterns
- VFS (Virtual File System) operations
- UI components and dialogs
- Testing strategies

### Commands

| Command | Description |
|---------|-------------|
| `/intellij:new-action` | Scaffold a new AnAction class |
| `/intellij:new-service` | Scaffold a new Service (Application/Project/Module) |
| `/intellij:new-tool-window` | Scaffold a new Tool Window |
| `/intellij:verify` | Run Plugin Verifier for compatibility checking |
| `/intellij:compat` | Check and update IDE version compatibility |

### Agent: Plugin Architect

A specialized agent for designing IntelliJ plugin architecture. Invoked when discussing:
- New plugin design
- Architecture decisions
- Code review for plugins
- Refactoring strategies

## Installation

### From Marketplace (Recommended)

```bash
# Add the marketplace
/plugin marketplace add masivotech/masivotech-claude-code

# Install the plugin
/plugin install intellij-plugin-dev@masivotech-plugins
```

### Local Installation

```bash
# Clone or download the plugin
git clone https://github.com/masivotech/masivotech-claude-code.git

# Add as local marketplace
/plugin marketplace add ./masivotech-claude-code

# Install
/plugin install intellij-plugin-dev@masivotech-plugins
```

### Global User Installation

Add to `~/.claude/settings.json`:

```json
{
  "enabledPlugins": {
    "intellij-plugin-dev": {
      "source": "~/.claude/plugins/intellij-plugin-dev"
    }
  }
}
```

## Usage Examples

### Creating a New Action

```
/intellij:new-action

> Create an action called "FormatCommitMessage" that formats the current
> commit message. Add it to the Vcs.MessageActionGroup with Ctrl+Shift+F shortcut.
```

### Designing Plugin Architecture

```
I want to build a plugin that shows real-time code metrics in a tool window.
It should update as the user types and show metrics like lines of code,
cyclomatic complexity, and method count for the current file.

[Claude will invoke the plugin-architect agent to design the architecture]
```

### Checking Compatibility

```
/intellij:compat

> Update my plugin to support IntelliJ IDEA 2024.2 through 2025.1
```

### Running Verification

```
/intellij:verify

> Run the plugin verifier and help me fix any compatibility issues
```

## Reference Documentation

The plugin includes comprehensive reference documentation in `skills/intellij-plugin-dev/references/`:

- **extension-points.md** - Catalog of common extension points
- **plugin-xml-guide.md** - Complete plugin.xml configuration reference
- **gradle-config.md** - Gradle IntelliJ Plugin 2.x setup
- **psi-patterns.md** - PSI navigation and manipulation patterns
- **vfs-patterns.md** - Virtual File System usage patterns
- **ui-components.md** - Tool windows, dialogs, notifications
- **testing-guide.md** - Unit, UI, and integration testing

## Supported IDEs

This plugin helps develop plugins for:

- IntelliJ IDEA (Community & Ultimate)
- Android Studio
- PyCharm
- WebStorm
- GoLand
- PhpStorm
- RubyMine
- CLion
- Rider

## Requirements

- Claude Code v2.0.12 or higher
- For plugin development: JDK 17+, Gradle 8.x

## Contributing

Contributions welcome! Please see the [GitHub repository](https://github.com/masivotech/masivotech-claude-code) for:

- Bug reports
- Feature requests
- Pull requests

## License

MIT License

## Author

masivotech
