---
description: Configure an existing Java project for jDeploy distribution. Detects build system, app type, configures JAR packaging, creates package.json, and sets up icons.
---

# jdeploy:setup

Configure an existing Java project for jDeploy distribution.

## Overview

This skill analyzes an existing Java project (Maven or Gradle) and configures it for distribution via jDeploy. It will:

1. Detect the build system and app characteristics
2. Configure executable JAR packaging
3. Create or update `package.json` with jDeploy configuration
4. Find and configure an application icon
5. Add GUI fallback for non-GUI apps
6. Optionally set up GitHub Actions workflow

## Instructions

### Step 1: Verify Prerequisites

Check that the project has a valid build configuration:

```bash
# Check for Maven
ls pom.xml 2>/dev/null && echo "Maven project detected"

# Check for Gradle
ls build.gradle build.gradle.kts 2>/dev/null && echo "Gradle project detected"
```

If neither exists, inform the user this skill requires an existing Java project with Maven or Gradle.

### Step 2: Detect App Characteristics

Run these detection checks to understand what kind of app this is:

**JavaFX detection:**
```bash
grep -r "import javafx\." src/ --include="*.java" 2>/dev/null | head -3
```

**CLI detection (argument parsing libraries):**
```bash
grep -r "import picocli\.\|import com.beust.jcommander\.\|import org.apache.commons.cli\.\|import org.kohsuke.args4j\." src/ --include="*.java" 2>/dev/null | head -3
```

**Service detection (web frameworks):**
```bash
grep -r "import org.springframework\.\|import io.quarkus\.\|import io.micronaut\.\|import io.vertx\.\|import io.javalin\." src/ --include="*.java" 2>/dev/null | head -3
```

**MCP Server detection:**
```bash
grep -r "import io.modelcontextprotocol\.\|@McpTool\|@Tool.*io.quarkiverse.mcp\|quarkus-mcp-server" src/ pom.xml build.gradle 2>/dev/null | head -3
```

**GUI detection (Swing/AWT):**
```bash
grep -r "import javax.swing\.\|import java.awt\.\|import androidx.compose\." src/ --include="*.java" --include="*.kt" 2>/dev/null | head -3
```

Based on results, classify the app:
- **GUI app**: Has Swing, JavaFX, or Compose imports
- **CLI app**: Has CLI library imports, no GUI
- **Service**: Has web framework imports
- **MCP Server**: Has MCP imports (also counts as CLI)

### Step 3: Identify Main Class

Find the main class:

```bash
# Search for main method
grep -r "public static void main" src/ --include="*.java" -l
```

Read the file(s) found to identify the fully-qualified class name.

### Step 4: Configure JAR Build

Check if the project already produces an executable JAR. If not, follow the instructions in CLAUDE.md section "jar-build" to configure:

- **Preferred**: JAR with dependencies in `lib/` directory
- **Alternative**: Shaded/fat JAR if already configured
- **Quarkus**: Enable uber-jar with `quarkus.package.jar.type=uber-jar`

### Step 5: Create package.json

Generate `package.json` based on detected characteristics. Use this template:

```json
{
  "name": "{{ app-name }}",
  "version": "1.0.0",
  "description": "",
  "bin": {
    "{{ app-name }}": "jdeploy-bundle/jdeploy.js"
  },
  "preferGlobal": true,
  "jdeploy": {
    "jdk": false,
    "javaVersion": "{{ java-version }}",
    "javafx": {{ true if JavaFX detected, else false }},
    "jar": "{{ path to JAR }}",
    "title": "{{ App Title }}"
  },
  "dependencies": {
    "command-exists-promise": "^2.0.2",
    "node-fetch": "2.6.7",
    "tar": "^4.4.8",
    "yauzl": "^2.10.0",
    "shelljs": "^0.8.4"
  },
  "files": ["jdeploy-bundle"],
  "license": "ISC"
}
```

**If CLI/Service/MCP detected**, add `jdeploy.commands`:

```json
"jdeploy": {
  ...
  "commands": {
    "{{ command-name }}": {
      "description": "{{ description }}"
    }
  }
}
```

**If MCP Server detected**, also add `jdeploy.ai.mcp`:

```json
"jdeploy": {
  ...
  "ai": {
    "mcp": {
      "command": "{{ command-name }}",
      "args": [],
      "defaultEnabled": false
    }
  }
}
```

### Step 6: Find and Configure Icon

Search for an existing icon:

```bash
find . -name "*.png" | xargs -I {} file {} | grep -i "PNG image" | head -10
```

Look for icons in:
- `src/main/resources/`
- `resources/`, `assets/`, `images/`
- For Compose: `app/src/main/res/mipmap-xxxhdpi/`

If a suitable square icon is found, copy it to `icon.png` in the project root.

### Step 7: Add GUI Fallback (for non-GUI apps)

If the app has NO GUI but has CLI/Service/MCP capabilities, inject a GUI fallback into the main class. Add this check at the start of `main()`:

```java
String mode = System.getProperty("jdeploy.mode", "gui");
if ("gui".equals(mode)) {
    javax.swing.SwingUtilities.invokeLater(() -> {
        javax.swing.JOptionPane.showMessageDialog(
            null,
            "{{ App Name }} v{{ version }}\n\nThis is a command-line tool.\nRun '{{ command }}' in a terminal for usage.",
            "About {{ App Name }}",
            javax.swing.JOptionPane.INFORMATION_MESSAGE
        );
        System.exit(0);
    });
    return;
}
```

For Quarkus apps, this must go in a `@QuarkusMain` class BEFORE calling `Quarkus.run()`.

### Step 8: Offer GitHub Workflow Setup

Ask the user if they want to set up GitHub Actions for automated releases. If yes, create `.github/workflows/jdeploy.yml` following the template in CLAUDE.md section "github-workflows".

### Step 9: Validate Setup

Build the project to verify the configuration works:

```bash
# Maven
mvn clean package

# Gradle
./gradlew build
```

Verify the JAR exists at the path specified in `package.json`.

## CLAUDE.md Reference

Consult these sections for detailed guidance:
- `prerequisites` - Project verification
- `jar-build` - JAR configuration for Maven/Gradle/Quarkus
- `detect-characteristics` - App type detection
- `package-json` - Full package.json schema
- `icon` - Icon requirements
- `gui-fallback` - GUI fallback implementation
- `github-workflows` - CI/CD setup

## Success Criteria

Setup is complete when:
1. `package.json` exists with valid jDeploy configuration
2. Project builds successfully and produces the specified JAR
3. JAR is executable (`java -jar` works)
4. Icon is configured (optional but recommended)
5. GUI fallback added (if applicable)
