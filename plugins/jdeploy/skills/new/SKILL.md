---
description: Create a new Java project from a jDeploy template. Supports Swing, JavaFX, CLI, Spring Boot, Quarkus, MCP servers, and more.
---

# jdeploy:new

Create a new Java project from a jDeploy template.

## Overview

This skill creates a new Java project using `npx jdeploy generate`. The generated project is pre-configured for jDeploy distribution with proper build configuration, package.json, and optionally GitHub integration.

## Instructions

### Step 1: Gather Project Details

Ask the user for the following information:

1. **Template** - What kind of app do you want to create?
   - `swing` - Traditional Swing desktop application (default)
   - `javafx` - JavaFX desktop application
   - `picocli` - Command-line tool with PicoCLI
   - `spring-boot-rest` - Spring Boot REST API service
   - `spring-boot-mcp-server` - Spring Boot MCP server
   - `quarkus-rest` - Quarkus REST API service
   - `quarkus-mcp-server` - Quarkus MCP server
   - `kotlin-multiplatform` - Kotlin multiplatform project
   - `fxgl` - FXGL game framework

2. **Fully-qualified main class name** - e.g., `com.example.myapp.MyApp`
   - This "magic argument" automatically derives:
     - Project name (from class name, hyphenated lowercase)
     - Group ID (from package prefix)
     - Artifact ID (same as project name)
     - Package name (from fully-qualified class)

3. **App title** - Human-readable name (e.g., "My Application")

4. **Parent directory** - Where to create the project (default: current directory)

5. **GitHub repository** (optional) - Format: `username/repo`
   - If provided, initializes git, creates the GitHub repo, and pushes

### Step 2: Run the Generate Command

Construct and execute the command:

```bash
npx jdeploy generate \
  -t <template> \
  -d <parent-directory> \
  --app-title="<App Title>" \
  [--github-repository=<username/repo>] \
  <fully.qualified.MainClass>
```

**Examples:**

Basic Swing app:
```bash
npx jdeploy generate -t swing --app-title="My Desktop App" com.example.myapp.MyApp
```

JavaFX app in specific directory:
```bash
npx jdeploy generate -t javafx -d ~/projects --app-title="My FX App" com.mycompany.fxapp.MainApp
```

MCP Server with GitHub:
```bash
npx jdeploy generate -t spring-boot-mcp-server --app-title="My MCP Tool" --github-repository=myuser/my-mcp-tool com.myuser.mcptool.MyMcpTool
```

CLI tool:
```bash
npx jdeploy generate -t picocli --app-title="My CLI" com.example.cli.MyCli
```

### Step 3: Verify Generation

After the command completes:

1. Navigate to the created project directory
2. Verify the structure looks correct:
   ```bash
   ls -la <project-dir>
   ```
3. Check that `package.json` was created with jDeploy config
4. Verify the build configuration (pom.xml or build.gradle)

### Step 4: Provide Next Steps

Tell the user:

1. **To build the project:**
   ```bash
   cd <project-name>
   # Maven
   mvn clean package
   # or Gradle
   ./gradlew build
   ```

2. **To run locally:**
   ```bash
   npx jdeploy run
   ```

3. **To install locally:**
   ```bash
   npx jdeploy install
   ```

4. **To publish:**
   - Use `/jdeploy:publish` skill
   - Or manually: `gh release create v1.0.0`

## Template Details

| Template | Build System | Description |
|----------|-------------|-------------|
| `swing` | Maven | Classic Swing desktop app with JFrame |
| `javafx` | Maven | Modern JavaFX desktop app |
| `picocli` | Maven | CLI tool with argument parsing |
| `spring-boot-rest` | Maven | Spring Boot REST service |
| `spring-boot-mcp-server` | Maven | Spring Boot MCP server for AI tools |
| `quarkus-rest` | Maven | Quarkus REST service |
| `quarkus-mcp-server` | Maven | Quarkus MCP server for AI tools |
| `kotlin-multiplatform` | Gradle | Kotlin multiplatform project |
| `fxgl` | Maven | Game development with FXGL |

## Command Reference

Full argument list for `npx jdeploy generate`:

| Argument | Short | Description |
|----------|-------|-------------|
| `--template-name` | `-t` | Template to use (default: "swing") |
| `--project-name` | `-n` | Project directory name |
| `--app-title` | | Human-readable app title |
| `--group-id` | `-g` | Maven group ID |
| `--artifact-id` | `-a` | Maven artifact ID |
| `--package-name` | `-pkg` | Java package name |
| `--main-class-name` | | Main class name (short, no package) |
| `--parent-directory` | `-d` | Directory to create project in |
| `--github-repository` | | GitHub repo (username/repo) |
| `--private-repository` | | Make GitHub repo private |

**Note:** The positional "magic argument" (fully-qualified class name) can infer most of these automatically.

## Error Handling

**"Template not found"**: The template name may be incorrect. List available templates:
```bash
npx jdeploy generate --help
```

**"Directory already exists"**: The project directory already exists. Either:
- Choose a different name
- Delete the existing directory
- Use `--use-existing-directory` flag (caution: may overwrite files)

**npm/npx not found**: Ensure Node.js is installed:
```bash
node --version
npm --version
```
