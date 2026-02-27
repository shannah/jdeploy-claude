---
description: Build and run a jDeploy project locally for testing. Uses npx jdeploy run to launch the app exactly as it would run when installed.
---

# jdeploy:run

Build and run a jDeploy project locally for testing.

## Overview

This skill builds the project and runs it using `npx jdeploy run`. This command:
- Installs the app locally if not already installed
- Runs it exactly as it would when installed by end users
- Uses the correct JRE version specified in package.json
- Handles JavaFX bundling automatically

**IMPORTANT**: Always use `npx jdeploy run` - never use `java -jar` directly, as that won't simulate the real installation environment.

## Instructions

### Step 1: Verify Project Setup

Check that the project has jDeploy configured:

```bash
ls package.json 2>/dev/null || echo "No package.json found"
```

If no `package.json` exists, suggest running `/jdeploy:setup` first.

### Step 2: Build the Project

Detect the build system and build:

**Maven:**
```bash
mvn clean package -DskipTests
```

**Gradle:**
```bash
./gradlew build -x test
```

If the build fails, report the error and stop.

### Step 3: Run with jDeploy

**Always use this command:**

```bash
npx jdeploy run
```

This will:
1. Install the app locally if not already installed
2. Use the JRE version specified in `jdeploy.javaVersion`
3. Bundle JavaFX if `jdeploy.javafx` is true
4. Launch the application exactly as end users would experience it

### Passing Arguments to the App

To pass arguments to your application:

```bash
npx jdeploy run -- --help
npx jdeploy run -- --config myconfig.json
npx jdeploy run -- arg1 arg2
```

Everything after `--` is passed to the Java application.

## Troubleshooting

**"JAR not found"**:
- Build the project first
- Check that `jdeploy.jar` path in `package.json` matches actual build output

**"Main class not found"**:
- Verify the JAR manifest has `Main-Class` attribute

**JavaFX issues**:
- Ensure `jdeploy.javafx` is set to `true` in `package.json`

## Quick Reference

```bash
# Build and run (Maven)
mvn clean package && npx jdeploy run

# Build and run (Gradle)
./gradlew build && npx jdeploy run

# Run with arguments
npx jdeploy run -- arg1 arg2
```
