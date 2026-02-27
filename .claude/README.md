# jDeploy Skills for Claude Code

A collection of Claude Code skills for creating and publishing Java applications with jDeploy.

## Available Skills

| Skill | Description |
|-------|-------------|
| `/jdeploy:setup` | Configure an existing Java project for jDeploy |
| `/jdeploy:new` | Create a new project from a template |
| `/jdeploy:run` | Build and run locally for testing |
| `/jdeploy:install` | Install locally (creates shortcuts, adds to PATH) |
| `/jdeploy:publish` | Publish to npm or GitHub |

## Quick Start

### New Project

```
/jdeploy:new
```

Creates a new Java project from a template. Choose from:
- Swing desktop app
- JavaFX desktop app
- CLI tool (picocli)
- Spring Boot REST service
- Spring Boot MCP server
- Quarkus REST service
- Quarkus MCP server
- And more...

### Existing Project

```
/jdeploy:setup
```

Analyzes your existing Java project and configures it for jDeploy:
- Detects app type (GUI, CLI, service, MCP server)
- Configures JAR packaging
- Creates `package.json`
- Finds/configures icon
- Adds GUI fallback for non-GUI apps
- Sets up GitHub Actions (optional)

### Development Workflow

```
/jdeploy:run      # Build and test locally
/jdeploy:install  # Install for full testing
/jdeploy:publish  # Ship it!
```

## Prerequisites

- **Java**: JDK 11 or later
- **Node.js**: For `npx jdeploy` commands
- **Build tool**: Maven or Gradle
- **GitHub CLI** (optional): For GitHub publishing (`gh`)

## How It Works

These skills use the jDeploy CLI via `npx jdeploy <command>`. No global installation required.

The skills follow instructions in [CLAUDE.md](../CLAUDE.md) for detailed configuration guidance.

## Project Configuration

Skills store minimal project preferences in `.jdeploy/config.json`:

```json
{
  "publishTarget": "github"
}
```

This remembers your publish target so you don't have to choose each time.

## Publish Targets

### GitHub (Recommended)

- Creates native installers for Windows, macOS, Linux
- Uses GitHub Actions for automated builds
- Users download from your releases page
- Supports MCP server auto-registration in AI tools

### npm

- Publishes to npm registry
- Users install with `npm install -g <package>`
- Good for CLI tools and developer utilities

## Templates

Available templates for `/jdeploy:new`:

| Template | Description |
|----------|-------------|
| `swing` | Traditional Swing desktop app |
| `javafx` | Modern JavaFX desktop app |
| `picocli` | CLI tool with argument parsing |
| `spring-boot-rest` | Spring Boot REST API |
| `spring-boot-mcp-server` | Spring Boot MCP server |
| `quarkus-rest` | Quarkus REST API |
| `quarkus-mcp-server` | Quarkus MCP server |
| `kotlin-multiplatform` | Kotlin multiplatform |
| `fxgl` | Game development |

## Links

- [jDeploy Website](https://www.jdeploy.com)
- [jDeploy GitHub](https://github.com/niceplume/jdeploy)
- [jDeploy Documentation](https://www.jdeploy.com/docs)
