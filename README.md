# jDeploy Plugin for Claude Code

A Claude Code plugin for creating and publishing cross-platform Java desktop applications with [jDeploy](https://www.jdeploy.com).

## Installation

### Step 1: Add the Marketplace

```
/plugin marketplace add shannah/jdeploy-claude
```

### Step 2: Install the Plugin

```
/plugin install jdeploy@jdeploy-marketplace
```

### Step 3: Verify Installation

The following skills should now be available:

- `/jdeploy:setup`
- `/jdeploy:new`
- `/jdeploy:run`
- `/jdeploy:install`
- `/jdeploy:publish`
- `/jdeploy:configure`

## Updating the Plugin

To get the latest version:

```
/plugin update jdeploy@jdeploy-marketplace
```

Or enable auto-updates:
1. Run `/plugin`
2. Go to **Marketplaces** tab
3. Select `jdeploy-marketplace`
4. Enable auto-update

---

## Available Skills

### `/jdeploy:new` - Create a New Project

Creates a new Java project from a template, fully configured for jDeploy.

**Usage:**
```
/jdeploy:new
```

Claude will ask you for:
- Project template (Swing, JavaFX, CLI, MCP server, etc.)
- Main class name (e.g., `com.example.myapp.MyApp`)
- App title
- Parent directory

**Available Templates:**

| Template | Description |
|----------|-------------|
| `swing` | Traditional Swing desktop app |
| `javafx` | Modern JavaFX desktop app |
| `picocli` | Command-line tool with argument parsing |
| `spring-boot-rest` | Spring Boot REST API |
| `spring-boot-mcp-server` | Spring Boot MCP server for AI tools |
| `quarkus-rest` | Quarkus REST API |
| `quarkus-mcp-server` | Quarkus MCP server for AI tools |
| `kotlin-multiplatform` | Kotlin multiplatform project |
| `fxgl` | FXGL game framework |

**Example:**
```
/jdeploy:new

> Template: javafx
> Main class: com.example.myapp.MyApp
> App title: My Desktop App
> Directory: ~/projects
```

---

### `/jdeploy:setup` - Configure Existing Project

Configures an existing Java project for jDeploy distribution.

**Usage:**
```
/jdeploy:setup
```

This will:
1. Detect your build system (Maven or Gradle)
2. Identify app type (GUI, CLI, service, MCP server)
3. Configure JAR packaging with dependencies
4. Create `package.json` with jDeploy settings
5. Find and configure an application icon
6. Add GUI fallback for non-GUI apps
7. Optionally set up GitHub Actions workflow

**When to use:**
- You have an existing Java project you want to distribute
- You want to create native installers for your app
- You want to publish to npm or GitHub

---

### `/jdeploy:run` - Build and Run Locally

Builds your project and runs it exactly as end users would experience it.

**Usage:**
```
/jdeploy:run
```

This will:
1. Build your project (Maven or Gradle)
2. Run via `npx jdeploy run`
3. Use the correct JRE version from your config
4. Bundle JavaFX if configured

**With arguments:**
```
/jdeploy:run
> Pass these args: --config myconfig.json
```

---

### `/jdeploy:install` - Install Locally

Installs your app locally for full testing, including desktop shortcuts and PATH integration.

**Usage:**
```
/jdeploy:install
```

This will:
1. Build your project
2. Run `npx jdeploy install`
3. Create desktop shortcuts
4. Add CLI commands to PATH

After installation, you can launch your app from the Start menu (Windows), Applications folder (macOS), or command line.

---

### `/jdeploy:publish` - Publish Your App

Publishes your application so users can install it.

**Usage:**
```
/jdeploy:publish
```

**Publish Targets:**

| Target | Description |
|--------|-------------|
| **GitHub** (Recommended) | Creates native installers (.exe, .dmg, .deb) via GitHub Actions. Users download from your releases page. |
| **npm** | Publishes to npm registry. Users install with `npm install -g <package>`. |

The skill remembers your choice in `.jdeploy/config.json` for future publishes.

**GitHub Publishing Flow:**
1. Verifies git repo and remote
2. Creates/verifies GitHub Actions workflow
3. Bumps version if needed
4. Creates a GitHub release
5. GitHub Actions builds native installers automatically

**npm Publishing Flow:**
1. Verifies npm login
2. Builds the project
3. Runs `npm publish`

---

### `/jdeploy:configure` - Configure App Settings

Configures various jDeploy settings for your application through a guided interface.

**Usage:**
```
/jdeploy:configure
```

**Available Configuration Options:**

| Option | Description |
|--------|-------------|
| Splash screen & icon | Set `splash.png`, `loading.png`, and `icon.png` |
| Java runtime | Version, JDK vs JRE, JavaFX, JetBrains Runtime |
| Commands & services | CLI commands, service controllers |
| Helper actions | System tray menu items |
| MCP server | AI tool integration |
| File associations | Register file extensions (`documentTypes`) |
| URL protocols | Custom URL schemes (`urlSchemes`) |
| Singleton mode | Single-instance with deep linking |
| Permissions | macOS permission requests |
| Run as admin | Elevated privileges |

**Example:**
```
/jdeploy:configure

> What would you like to configure?
> [x] Singleton mode
> [x] URL protocols

# Claude guides you through each selected option
```

---

## Example Workflow

### Creating a New Desktop App

```
# Create a new JavaFX project
/jdeploy:new

# Test it locally
/jdeploy:run

# Install locally for full testing
/jdeploy:install

# Publish to GitHub
/jdeploy:publish
```

### Converting an Existing Project

```
# Configure jDeploy
/jdeploy:setup

# Test the configuration
/jdeploy:run

# Publish when ready
/jdeploy:publish
```

---

## What is jDeploy?

jDeploy lets you deploy Java applications as native desktop apps without requiring Java on target machines. It:

- Creates native installers for Windows (.exe), macOS (.dmg), and Linux (.deb)
- Bundles the appropriate JRE automatically
- Supports GUI apps, CLI tools, background services, and MCP servers
- Publishes via npm or GitHub Releases
- Auto-registers MCP servers with AI tools (Claude Desktop, VS Code, Cursor, etc.)

---

## Prerequisites

- **Java**: JDK 11 or later
- **Node.js**: For `npx jdeploy` commands
- **Build tool**: Maven or Gradle
- **GitHub CLI** (optional): For GitHub publishing (`gh`)

---

## Links

- [jDeploy Website](https://www.jdeploy.com)
- [jDeploy Documentation](https://www.jdeploy.com/docs)
- [jDeploy GitHub](https://github.com/niceplume/jdeploy)

---

## Plugin Development

### Project Structure

```
jdeploy-claude/
├── .claude-plugin/
│   └── marketplace.json        # Marketplace catalog
├── plugins/
│   └── jdeploy/
│       ├── .claude-plugin/
│       │   └── plugin.json     # Plugin manifest
│       └── skills/
│           ├── setup/SKILL.md
│           ├── new/SKILL.md
│           ├── run/SKILL.md
│           ├── install/SKILL.md
│           ├── publish/SKILL.md
│           └── configure/SKILL.md
├── CLAUDE.md                   # Detailed setup reference
└── README.md
```

### Contributing

1. Fork this repository
2. Make changes to skill files in `plugins/jdeploy/skills/`
3. Test locally with `/plugin marketplace add ./path/to/jdeploy-claude`
4. Submit a pull request
