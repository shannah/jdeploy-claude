---
description: Configure jDeploy app settings including splash screens, icons, commands, services, MCP servers, file associations, URL protocols, singleton mode, permissions, and Java runtime options.
---

# jdeploy:configure

Configure jDeploy application settings.

## Overview

This skill helps configure various jDeploy settings for your application. It detects your current configuration and presents relevant options based on your app type.

## Instructions

### Step 1: Read Current Configuration

First, read the existing `package.json` to understand current state:

```bash
cat package.json 2>/dev/null || echo "No package.json found"
```

If no `package.json` exists, suggest running `/jdeploy:setup` first.

Parse the `jdeploy` section to determine:
- What's already configured
- App type (GUI, CLI, Service, MCP)
- Whether commands exist
- Whether service controller is configured

### Step 2: Present Configuration Options

Based on context, present relevant options to the user. Use AskUserQuestion to let them choose what to configure:

**Always show:**
1. Splash screen & loading image
2. Application icon
3. Java runtime (version, JDK, JavaFX)
4. Permissions

**Show if app has or could have commands:**
5. Commands & services
6. Helper (system tray) actions (only if service controller exists)

**Show if app is CLI/service type:**
7. MCP server integration

**Show for desktop apps:**
8. File associations (documentTypes)
9. URL protocols (urlSchemes)
10. Singleton mode

Allow the user to select multiple options to configure in one session.

---

## Configuration Sections

### Section: Splash Screen & Loading Image

jDeploy supports two types of startup images:

1. **Splash screen** (`splash.png`) - Shown immediately when app launches, before JVM starts
2. **Loading image** (`loading.png`) - Shown while JVM initializes (optional, falls back to splash)

**Requirements:**
- Format: PNG
- Location: Project root (same directory as `package.json`)
- Recommended size: 600x400 or similar (not too large)

**Steps:**

1. Check for existing splash/loading images:
   ```bash
   ls -la splash.png loading.png 2>/dev/null
   ```

2. If user wants to set a splash screen, help them:
   - Search for candidate images in the project
   - Copy to `splash.png` in project root
   - Optionally create a separate `loading.png`

3. No `package.json` changes needed - jDeploy auto-detects these files.

**Example search:**
```bash
find . -name "*.png" -size +10k | head -20
```

---

### Section: Application Icon

The application icon is used for desktop shortcuts, taskbar, installers, etc.

**Requirements:**
- Format: PNG
- Name: `icon.png` in project root
- Recommended size: 512x512 or 1024x1024 (square)

**Steps:**

1. Check for existing icon:
   ```bash
   ls -la icon.png 2>/dev/null && file icon.png
   ```

2. Search for candidate icons:
   ```bash
   find . -name "*icon*.png" -o -name "*logo*.png" | head -10
   ```

3. Verify the icon is square:
   ```bash
   file candidate.png
   ```

4. Copy to project root:
   ```bash
   cp path/to/icon.png ./icon.png
   ```

---

### Section: Java Runtime Configuration

Configure which Java runtime jDeploy bundles with your app.

**Properties in `package.json` → `jdeploy`:**

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `javaVersion` | string | `"17"` | Java major version (8, 11, 17, 21, etc.) |
| `jdk` | boolean | `false` | Bundle full JDK instead of JRE |
| `javafx` | boolean | `false` | Include JavaFX modules |
| `javafxVersion` | string | (auto) | Specific JavaFX version (optional) |
| `jdkProvider` | string | `"zulu"` | JDK provider: `"zulu"` or `"jbr"` (JetBrains Runtime) |
| `jbrVariant` | string | `"standard"` | JBR variant: `"standard"`, `"jcef"`, `"sdk"`, `"sdk_jcef"` |

**Steps:**

1. Ask user what they need:
   - What Java version does the app require?
   - Does it need JavaFX?
   - Does it need full JDK (for tools like `javac`, `jlink`, etc.)?
   - Does it need JCEF (embedded Chromium browser)?

2. Update `package.json`:

```json
{
  "jdeploy": {
    "javaVersion": "21",
    "jdk": false,
    "javafx": true
  }
}
```

**For JetBrains Runtime with JCEF:**
```json
{
  "jdeploy": {
    "javaVersion": "21",
    "jdkProvider": "jbr",
    "jbrVariant": "jcef"
  }
}
```

**Detection helpers:**

Check for JavaFX usage:
```bash
grep -r "import javafx\." src/ --include="*.java" 2>/dev/null | head -3
```

Check for JDK tool usage:
```bash
grep -r "ToolProvider\|JavaCompiler\|ProcessBuilder.*javac" src/ --include="*.java" 2>/dev/null | head -3
```

---

### Section: Commands & Services

Commands create CLI executables that get added to the user's PATH when installed.

**Properties in `package.json` → `jdeploy.commands`:**

Each command is a key-value pair where the key is the command name:

```json
{
  "jdeploy": {
    "commands": {
      "myapp-cli": {
        "description": "Run MyApp from command line",
        "args": ["-Dmode=cli"],
        "implements": []
      }
    }
  }
}
```

**Command properties:**

| Property | Type | Required | Description |
|----------|------|----------|-------------|
| `description` | string | No | Human-readable description |
| `args` | array | No | Arguments passed to Java app when command is invoked |
| `implements` | array | No | Roles: `"service_controller"`, `"updater"`, `"launcher"` |

**Service Controller:**

To make a command act as a system service controller, add `"service_controller"` to `implements`:

```json
{
  "jdeploy": {
    "commands": {
      "myservice": {
        "description": "My background service",
        "implements": ["service_controller"]
      }
    }
  }
}
```

This enables subcommands:
- `myservice service install` - Install as system service
- `myservice service start` - Start the service
- `myservice service stop` - Stop the service
- `myservice service status` - Check service status
- `myservice service uninstall` - Remove system service

**Steps:**

1. Ask user:
   - What command name(s) do they want?
   - Should any command run as a system service?
   - Any special arguments needed?

2. Update `package.json` with the commands configuration.

3. **Important**: Also update the `bin` field to include new commands:

```json
{
  "bin": {
    "myapp": "jdeploy-bundle/jdeploy.js",
    "myapp-cli": "jdeploy-bundle/jdeploy.js"
  }
}
```

---

### Section: Helper (System Tray) Actions

The Background Helper provides system tray integration with custom menu actions. Actions typically use custom URL schemes to communicate with your app.

**Prerequisite:** Works best with apps that have `urlSchemes` configured and `singleton` mode enabled.

**Properties in `package.json` → `jdeploy.helper`:**

```json
{
  "jdeploy": {
    "urlSchemes": ["myapp"],
    "singleton": true,
    "helper": {
      "actions": [
        {
          "label": "Open Dashboard",
          "description": "Open the main application window",
          "url": "myapp://open/dashboard"
        },
        {
          "label": "View Logs",
          "url": "myapp://open/logs"
        }
      ]
    }
  }
}
```

**Action properties:**

| Property | Type | Required | Description |
|----------|------|----------|-------------|
| `label` | string | Yes | Text displayed in the menu item |
| `description` | string | No | Tooltip text shown on hover |
| `url` | string | Yes | URL to open when clicked (typically custom scheme) |

**Steps:**

1. Check if urlSchemes and singleton are configured (recommended for helper actions)

2. Ask user what tray actions they need:
   - Open main window?
   - View logs?
   - Access settings?
   - Custom actions?

3. Update `package.json` with helper configuration.

---

### Section: MCP Server Integration

Configure the app as an MCP (Model Context Protocol) server for AI tool integration.

**Properties in `package.json` → `jdeploy.ai.mcp`:**

```json
{
  "jdeploy": {
    "commands": {
      "my-mcp-server": {
        "description": "MCP server for AI tools"
      }
    },
    "ai": {
      "mcp": {
        "command": "my-mcp-server",
        "args": ["--stdio"],
        "defaultEnabled": false
      }
    }
  }
}
```

**MCP properties:**

| Property | Type | Required | Description |
|----------|------|----------|-------------|
| `command` | string | Yes | Must match a key in `jdeploy.commands` |
| `args` | array | No | Additional args when invoked as MCP server |
| `defaultEnabled` | boolean | No | Auto-enable during installation (default: false) |

**Steps:**

1. Check if a command exists that could serve as MCP entry point:
   ```bash
   node -p "Object.keys(require('./package.json').jdeploy?.commands || {})"
   ```

2. If no commands exist, create one first (see Commands section).

3. Ask user:
   - Which command should be the MCP entry point?
   - Any additional arguments for MCP mode?
   - Should it be enabled by default during install?

4. Update `package.json` with the `ai.mcp` configuration.

5. **Reminder**: MCP servers communicate via stdin/stdout, so ensure the app:
   - Doesn't write to stdout except for MCP protocol
   - Disables console logging (use file logging instead)

---

### Section: File Associations (documentTypes)

Register file extensions to open with your application.

**Properties in `package.json` → `jdeploy.documentTypes`:**

```json
{
  "jdeploy": {
    "documentTypes": [
      {
        "extension": "myformat",
        "mimetype": "application/x-myformat",
        "editor": true,
        "custom": true
      }
    ]
  }
}
```

**Document type properties:**

| Property | Type | Required | Description |
|----------|------|----------|-------------|
| `extension` | string | Yes | File extension without dot (e.g., `"txt"`) |
| `mimetype` | string | Yes | MIME type for the file |
| `editor` | boolean | No | `true` if app can edit (not just view) files |
| `custom` | boolean | No | `true` for custom mimetypes (Linux registration) |

**Directory associations** (for IDE-style apps):

```json
{
  "jdeploy": {
    "documentTypes": [
      {
        "type": "directory",
        "role": "Editor",
        "description": "Open folder as project"
      }
    ]
  }
}
```

**Steps:**

1. Ask user what file types the app should handle:
   - Extension(s)?
   - MIME type?
   - Can it edit or just view?
   - Is this a custom mimetype?

2. Update `package.json` with documentTypes array.

3. **Tip**: Enable singleton mode (`"singleton": true`) so double-clicking files activates the existing window rather than launching a new instance.

**Example - handling opened files in Java:**
```java
public static void main(String[] args) {
    if (args.length > 0) {
        String filePath = args[0];
        // Open the file
    }
}
```

---

### Section: URL Protocols (urlSchemes)

Register custom URL protocols (e.g., `myapp://action`) that open your application.

**Properties in `package.json` → `jdeploy.urlSchemes`:**

```json
{
  "jdeploy": {
    "urlSchemes": ["myapp", "myapp-alt"]
  }
}
```

The `urlSchemes` property is a simple array of scheme strings (without `://`).

**Steps:**

1. Ask user what URL scheme(s) they want:
   - Scheme name (e.g., `myapp` for `myapp://...` URLs)?

2. Update `package.json` with urlSchemes array.

3. **Tip**: Enable singleton mode (`"singleton": true`) so clicking URLs activates the existing window rather than launching a new instance.

**Example - handling URL in Java:**
```java
public static void main(String[] args) {
    if (args.length > 0 && args[0].startsWith("myapp://")) {
        String url = args[0];
        // Parse and handle the URL
    }
}
```

---

### Section: Singleton Mode

Ensure only one instance of the application runs at a time. When a second instance is launched, it forwards files/URIs to the running instance via IPC.

**Requirements:**
1. Set `"singleton": true` in package.json
2. Add `jdeploy-desktop-lib` dependency to receive forwarded events
3. Register an open handler in your app code

**Step 1: Enable in package.json**

```json
{
  "jdeploy": {
    "singleton": true,
    "urlSchemes": ["myapp"]
  }
}
```

**Step 2: Add dependency**

**For Swing apps (Maven):**
```xml
<dependency>
    <groupId>ca.weblite</groupId>
    <artifactId>jdeploy-desktop-lib-swing</artifactId>
    <version>1.0.2</version>
</dependency>
```

**For Swing apps (Gradle):**
```gradle
implementation 'ca.weblite:jdeploy-desktop-lib-swing:1.0.2'
```

**For JavaFX apps (Maven):**
```xml
<dependency>
    <groupId>ca.weblite</groupId>
    <artifactId>jdeploy-desktop-lib-javafx</artifactId>
    <version>1.0.2</version>
</dependency>
```

**For JavaFX apps (Gradle):**
```gradle
implementation 'ca.weblite:jdeploy-desktop-lib-javafx:1.0.2'
```

**Step 3: Register handler in code**

**Swing:**
```java
import ca.weblite.jdeploy.app.swing.JDeploySwingApp;
import ca.weblite.jdeploy.app.JDeployOpenHandler;

public class MyApp {
    public static void main(String[] args) {
        JDeploySwingApp.setOpenHandler(new JDeployOpenHandler() {
            public void openFiles(List<File> files) {
                for (File file : files) {
                    openDocument(file);
                }
            }
            public void openURIs(List<URI> uris) {
                for (URI uri : uris) {
                    handleDeepLink(uri);
                }
            }
            public void appActivated() {
                // Bring window to front
            }
        });
        // ... rest of app
    }
}
```

**JavaFX:**
```java
import ca.weblite.jdeploy.app.javafx.JDeployFXApp;
import ca.weblite.jdeploy.app.JDeployOpenHandler;

public class MyApp extends Application {
    public static void main(String[] args) {
        JDeployFXApp.initialize();  // REQUIRED before launch() on macOS
        launch(args);
    }

    @Override
    public void start(Stage primaryStage) {
        JDeployFXApp.setOpenHandler(new JDeployOpenHandler() {
            public void openFiles(List<File> files) { /* handle files */ }
            public void openURIs(List<URI> uris) { /* handle URIs */ }
            public void appActivated() { /* bring window to front */ }
        });
        // ... rest of app
    }
}
```

**Steps:**

1. Check if jdeploy-desktop-lib is already a dependency:
   ```bash
   grep -r "jdeploy-desktop-lib" pom.xml build.gradle build.gradle.kts 2>/dev/null
   ```

2. Ask if it's a Swing or JavaFX app to choose the right artifact.

3. Add the dependency to the build file.

4. Guide user to add handler code to their main class.

5. Update `package.json` to enable singleton mode.

---

### Section: Permissions

Configure what permissions the app requests on different platforms (currently macOS only).

**Properties in `package.json` → `jdeploy.permissions`:**

```json
{
  "jdeploy": {
    "permissions": [
      {
        "name": "camera",
        "description": "Camera access is required for video calls"
      },
      {
        "name": "microphone",
        "description": "Microphone access is required for voice recording"
      }
    ]
  }
}
```

**Permission properties:**

| Property | Type | Required | Description |
|----------|------|----------|-------------|
| `name` | string | Yes | Permission name (see table below) |
| `description` | string | Yes | Why the app needs this permission (shown to user) |

**Available permission names:**

| Permission | Description |
|------------|-------------|
| `camera` | Access to the camera |
| `microphone` | Access to the microphone |
| `location` | Access to user's location |
| `location_when_in_use` | Location access when app is in use |
| `location_always` | Location access at all times |
| `contacts` | Access to contacts |
| `calendars` | Access to calendars |
| `reminders` | Access to reminders |
| `photos` | Access to photo library |
| `photos_add` | Add photos to library |
| `bluetooth` | Bluetooth access |
| `bluetooth_always` | Bluetooth access at all times |
| `speech_recognition` | Speech recognition |
| `desktop_folder` | Access to Desktop folder |
| `documents_folder` | Access to Documents folder |
| `downloads_folder` | Access to Downloads folder |
| `network_volumes` | Access to network volumes |
| `removable_volumes` | Access to removable volumes |
| `local_network` | Access to local network |

**Steps:**

1. Ask user what permissions the app needs.

2. For each permission, ask for a description explaining why it's needed.

3. Update `package.json` with permissions array.

---

### Section: Run as Administrator

Configure the application to run with elevated privileges.

**Property in `package.json` → `jdeploy.runAsAdministrator`:**

| Value | Description |
|-------|-------------|
| `"disabled"` | (Default) Normal user privileges |
| `"allowed"` | Generates both standard and admin launchers |
| `"required"` | All launchers use elevation |

```json
{
  "jdeploy": {
    "runAsAdministrator": "allowed"
  }
}
```

**Steps:**

1. Ask if the app needs elevated privileges:
   - Never needs admin? → `"disabled"`
   - Sometimes needs admin? → `"allowed"`
   - Always needs admin? → `"required"`

2. Update `package.json` with the setting.

---

### Section: Windows Installation Directory

Configure a custom installation directory on Windows to avoid Windows Defender warnings.

**Property in `package.json` → `jdeploy.winAppDir`:**

```json
{
  "jdeploy": {
    "winAppDir": "AppData\\Local\\Programs"
  }
}
```

This path is relative to `%USERPROFILE%`. Default is `"jdeploy"` (installs to `%USERPROFILE%\jdeploy\apps`).

---

## Handling Multiple Configurations

When the user selects multiple options to configure:

1. Process them in a logical order:
   - Java runtime first (foundational)
   - Commands (may be prerequisite for others)
   - URL schemes and singleton (often used together)
   - Other settings

2. After each section, show a summary of changes made.

3. At the end, show the complete updated `jdeploy` configuration.

## Validation

After making changes, validate the configuration:

```bash
# Verify package.json is valid JSON
node -e "require('./package.json')"

# Show the jdeploy config
node -p "JSON.stringify(require('./package.json').jdeploy, null, 2)"
```

## Quick Reference

| Configuration | package.json location | Files/Dependencies needed |
|--------------|----------------------|---------------------------|
| Splash screen | (auto-detected) | `splash.png` |
| Loading image | (auto-detected) | `loading.png` |
| Icon | (auto-detected) | `icon.png` |
| Java version | `jdeploy.javaVersion` | - |
| Full JDK | `jdeploy.jdk` | - |
| JavaFX | `jdeploy.javafx` | - |
| JDK provider | `jdeploy.jdkProvider` | - |
| Commands | `jdeploy.commands` | Update `bin` field too |
| Service controller | `jdeploy.commands[name].implements` | - |
| Helper actions | `jdeploy.helper.actions` | - |
| MCP server | `jdeploy.ai.mcp` | Requires command |
| File associations | `jdeploy.documentTypes` | - |
| URL protocols | `jdeploy.urlSchemes` | - |
| Singleton mode | `jdeploy.singleton` | `jdeploy-desktop-lib-*` |
| Permissions | `jdeploy.permissions` | - |
| Run as admin | `jdeploy.runAsAdministrator` | - |
| Windows install dir | `jdeploy.winAppDir` | - |
