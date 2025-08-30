# jDeploy Project Templates

This directory contains example project configurations for different types of Java applications using jDeploy.

## Available Templates

### Basic Templates
- **`basic-maven/`** - Simple Maven project with executable JAR
- **`basic-gradle/`** - Simple Gradle project with executable JAR

### UI Framework Templates  
- **`javafx/`** - JavaFX desktop application
- **`swing/`** - Traditional Swing desktop application
- **`spring-boot-swing/`** - Spring Boot application with Swing UI

### Advanced Templates
- **`gradle-modular/`** - Modular Gradle project using Java modules

## Using Templates

Each template contains:
- **`pom.xml`** or **`build.gradle`** - Build configuration with jDeploy-compatible JAR setup
- **`package.json`** - jDeploy configuration for the project type
- **`src/`** - Example source code structure
- **`README.md`** - Template-specific setup instructions

## Template Features

All templates are configured to:
1. Build executable JARs with proper classpath configuration
2. Generate dependencies in `lib/` directory (preferred) or use shaded JARs
3. Include appropriate jDeploy `package.json` configuration
4. Work with both desktop and command-line jDeploy installations