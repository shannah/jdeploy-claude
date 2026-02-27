# jdeploy:run

Build and run a jDeploy project locally for testing.

## Overview

This skill builds the project and runs it using jDeploy's local runner. This is the fastest way to test your application during development.

## Instructions

### Step 1: Verify Project Setup

Check that the project has jDeploy configured:

```bash
ls package.json 2>/dev/null || echo "No package.json found"
```

If no `package.json` exists, suggest running `/jdeploy:setup` first.

### Step 2: Detect Build System

```bash
# Check for Maven
if [ -f pom.xml ]; then echo "maven"; fi

# Check for Gradle
if [ -f build.gradle ] || [ -f build.gradle.kts ]; then echo "gradle"; fi
```

### Step 3: Build the Project

**Maven:**
```bash
mvn clean package -DskipTests
```

**Gradle:**
```bash
./gradlew build -x test
```

If the build fails, report the error and stop.

### Step 4: Run with jDeploy

```bash
npx jdeploy run
```

This will:
1. Locate the JAR specified in `package.json`
2. Download/use the appropriate JRE for the configured Java version
3. Launch the application

### Alternative: Direct JAR Execution

If `npx jdeploy run` isn't working, you can run the JAR directly:

```bash
# Read JAR path from package.json
JAR_PATH=$(node -p "require('./package.json').jdeploy.jar")
java -jar "$JAR_PATH"
```

## Options

The `jdeploy run` command accepts these options:

| Option | Description |
|--------|-------------|
| `--` | Pass remaining arguments to the Java application |

**Example with app arguments:**
```bash
npx jdeploy run -- --help
npx jdeploy run -- --config myconfig.json
```

## Troubleshooting

**"JAR not found"**:
- Build the project first
- Check that `jdeploy.jar` path in `package.json` matches actual output

**"Main class not found"**:
- Verify the JAR manifest has `Main-Class` attribute
- Check: `unzip -p target/myapp.jar META-INF/MANIFEST.MF | grep Main-Class`

**"Java version mismatch"**:
- jDeploy downloads the JRE specified in `jdeploy.javaVersion`
- If you need a different version, update `package.json`

**JavaFX issues**:
- Ensure `jdeploy.javafx` is set to `true` in `package.json`
- jDeploy will bundle JavaFX modules automatically

## Quick Reference

```bash
# Build and run (Maven)
mvn clean package && npx jdeploy run

# Build and run (Gradle)
./gradlew build && npx jdeploy run

# Run with arguments
npx jdeploy run -- arg1 arg2

# Skip build, just run
npx jdeploy run
```
