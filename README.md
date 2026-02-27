# jDeploy Plugin Marketplace for Claude Code

A Claude Code plugin marketplace for creating and publishing cross-platform Java desktop applications with [jDeploy](https://www.jdeploy.com).

## Installation

Add the marketplace and install the plugin:

```
/plugin marketplace add shannah/jdeploy-claude
/plugin install jdeploy@jdeploy-marketplace
```

## Available Skills

Once installed, you have access to these skills:

| Skill | Description |
|-------|-------------|
| `/jdeploy:setup` | Configure an existing Java project for jDeploy |
| `/jdeploy:new` | Create a new project from a template |
| `/jdeploy:run` | Build and run locally for testing |
| `/jdeploy:install` | Install locally (creates shortcuts, adds to PATH) |
| `/jdeploy:publish` | Publish to npm or GitHub |

## Quick Start

### Create a New Project

```
/jdeploy:new
```

Choose from templates:
- `swing` - Swing desktop app
- `javafx` - JavaFX desktop app
- `picocli` - CLI tool
- `spring-boot-mcp-server` - MCP server for AI tools
- `quarkus-mcp-server` - Quarkus MCP server
- And more...

### Configure an Existing Project

```
/jdeploy:setup
```

This will:
- Detect your build system (Maven/Gradle)
- Identify app type (GUI, CLI, service, MCP server)
- Configure JAR packaging
- Create `package.json` with jDeploy settings
- Find and configure an icon
- Add GUI fallback for non-GUI apps
- Optionally set up GitHub Actions

### Development Workflow

```
/jdeploy:run      # Build and test locally
/jdeploy:install  # Install for full testing
/jdeploy:publish  # Ship it!
```

## What is jDeploy?

jDeploy lets you deploy Java applications as native desktop apps without requiring Java on target machines. It:

- Creates native installers for Windows (.exe), macOS (.dmg), and Linux (.deb)
- Bundles the appropriate JRE automatically
- Supports GUI apps, CLI tools, services, and MCP servers
- Publishes via npm or GitHub Releases
- Auto-registers MCP servers with AI tools (Claude Desktop, VS Code, Cursor, etc.)

## Prerequisites

- **Java**: JDK 11 or later
- **Node.js**: For `npx jdeploy` commands
- **Build tool**: Maven or Gradle
- **GitHub CLI** (optional): For GitHub publishing

## Project Structure

```
jdeploy-claude/
├── .claude-plugin/
│   └── marketplace.json      # Marketplace catalog
├── plugins/
│   └── jdeploy/
│       ├── .claude-plugin/
│       │   └── plugin.json   # Plugin manifest
│       └── commands/
│           ├── setup.md
│           ├── new.md
│           ├── run.md
│           ├── install.md
│           └── publish.md
├── CLAUDE.md                 # Detailed setup instructions
└── README.md
```

## Links

- [jDeploy Website](https://www.jdeploy.com)
- [jDeploy Documentation](https://www.jdeploy.com/docs)
- [jDeploy GitHub](https://github.com/niceplume/jdeploy)
