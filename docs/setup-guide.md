# jDeploy Setup Guide

This guide walks through setting up a Java project to work with jDeploy.

## Step 1: Verify Project Prerequisites

Before setting up jDeploy, ensure your Java project:

1. **Builds successfully** with Maven or Gradle
2. **Has a main class** that serves as the application entry point
3. **Produces executable output** (we'll configure this if needed)

### Check Your Build System

**Maven Project:** Look for `pom.xml` in project root
```bash
mvn clean compile
```

**Gradle Project:** Look for `build.gradle` in project root  
```bash
./gradlew build
```

## Step 2: Configure Executable JAR Build

### Option A: JAR with Dependencies in lib/ Directory (Recommended)

This approach allows jDeploy to optimize the published bundle by filtering unnecessary dependencies.

#### Maven Configuration

Add these plugins to your `pom.xml`:

```xml
<build>
    <plugins>
        <!-- Copy dependencies to lib/ directory -->
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-dependency-plugin</artifactId>
            <version>3.2.0</version>
            <executions>
                <execution>
                    <id>copy-dependencies</id>
                    <phase>package</phase>
                    <goals>
                        <goal>copy-dependencies</goal>
                    </goals>
                    <configuration>
                        <outputDirectory>${project.build.directory}/lib</outputDirectory>
                    </configuration>
                </execution>
            </executions>
        </plugin>
        
        <!-- Configure JAR with classpath -->
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-jar-plugin</artifactId>
            <version>3.2.2</version>
            <configuration>
                <archive>
                    <manifest>
                        <addClasspath>true</addClasspath>
                        <classpathPrefix>lib/</classpathPrefix>
                        <mainClass>com.example.MainClass</mainClass>
                    </manifest>
                </archive>
            </configuration>
        </plugin>
    </plugins>
</build>
```

**Replace `com.example.MainClass` with your actual main class.**

Build and verify:
```bash
mvn clean package
ls target/      # Should see your-app.jar and lib/ directory
java -jar target/your-app.jar  # Should run successfully
```

#### Gradle Configuration

Add to your `build.gradle`:

```gradle
plugins {
    id 'application'
    id 'java'
}

application {
    mainClass = 'com.example.MainClass'
}

// Task to copy dependencies
task copyDependencies(type: Copy) {
    from configurations.runtimeClasspath
    into "$buildDir/libs/lib"
}

// Configure JAR to depend on copied dependencies
jar {
    dependsOn copyDependencies
    manifest {
        attributes(
            'Main-Class': application.mainClass,
            'Class-Path': configurations.runtimeClasspath.collect { "lib/" + it.getName() }.join(' ')
        )
    }
}
```

Build and verify:
```bash
./gradlew build
ls build/libs/  # Should see your-app.jar and lib/ directory  
java -jar build/libs/your-app.jar  # Should run successfully
```

### Option B: Shaded/Fat JAR (If Already Configured)

If your project already produces a shaded JAR (all dependencies included), you can keep that configuration:

**Maven:** Keep existing `maven-shade-plugin` configuration
**Gradle:** Keep existing `shadow` plugin configuration

## Step 3: Create or Update package.json

Create `package.json` in your project root with the following structure:

### Basic Configuration

```json
{
  "name": "my-java-app",
  "version": "1.0.0",
  "description": "My Java application deployed with jDeploy",
  "bin": {
    "my-java-app": "jdeploy-bundle/jdeploy.js"
  },
  "dependencies": {
    "shelljs": "^0.8.4"
  },
  "jdeploy": {
    "jar": "target/my-app-1.0.jar",
    "javaVersion": "11",
    "title": "My Java Application"
  }
}
```

### Configuration Fields

#### Required Fields:
- **`name`**: Unique NPM package name (lowercase, hyphens allowed)
- **`bin`**: Maps command name to jDeploy launcher
- **`dependencies`**: Must include `"shelljs": "^0.8.4"`
- **`jdeploy.jar`**: Path to your executable JAR file
- **`jdeploy.javaVersion`**: Java version required ("8", "11", "17", etc.)
- **`jdeploy.title`**: Human-readable application name

#### Optional Fields:
- **`jdeploy.jdk`**: Set to `true` if full JDK is required (default: false)
- **`jdeploy.javafx`**: Set to `true` for JavaFX applications (default: false)
- **`jdeploy.args`**: Array of JVM arguments

### Configuration Examples by Project Type

#### Standard Maven Project:
```json
"jdeploy": {
  "jar": "target/myapp-1.0.jar",
  "javaVersion": "11",
  "title": "My Application"
}
```

#### Standard Gradle Project:
```json
"jdeploy": {
  "jar": "build/libs/myapp-1.0.jar", 
  "javaVersion": "11",
  "title": "My Application"
}
```

#### Shaded JAR (Maven):
```json
"jdeploy": {
  "jar": "target/myapp-1.0-jar-with-dependencies.jar",
  "javaVersion": "11",
  "title": "My Application"
}
```

#### JavaFX Application:
```json
"jdeploy": {
  "jar": "target/myapp-1.0.jar",
  "javaVersion": "11", 
  "javafx": true,
  "title": "My JavaFX App"
}
```

#### Application with JVM Arguments:
```json
"jdeploy": {
  "jar": "target/myapp-1.0.jar",
  "javaVersion": "11",
  "title": "My Application",
  "args": [
    "-Xmx2G",
    "-Dapp.config=production"
  ]
}
```

## Step 4: Build and Test Configuration

1. **Build your Java project:**
   ```bash
   # Maven
   mvn clean package
   
   # Gradle  
   ./gradlew build
   ```

2. **Verify JAR is executable:**
   ```bash
   java -jar target/your-app.jar  # Maven
   java -jar build/libs/your-app.jar  # Gradle
   ```

3. **Check package.json paths:**
   - Ensure the `jar` path in package.json matches your actual build output
   - Verify the JAR file exists after building

## Step 5: Optional GitHub Workflows

To automatically create app bundles when you push code or tags, create `.github/workflows/jdeploy.yml`:

```yaml
name: jDeploy CI

on:
  push:
    branches:
      - '**'
      - '!gh-pages'
    tags:
      - '**'

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
    - uses: actions/checkout@v4
    
    - name: Set up JDK
      uses: actions/setup-java@v4
      with:
        java-version: '21'
        distribution: 'temurin'
    
    # For Gradle projects
    - name: Make gradlew executable
      run: chmod +x ./gradlew
    
    - name: Build with Gradle
      run: ./gradlew build
      
    # OR for Maven projects:
    # - name: Build with Maven
    #   run: mvn clean package
    
    - name: Build App Installer Bundles
      uses: actions/jdeploy@v1.0.0
      with:
        github_token: ${{ secrets.GITHUB_TOKEN }}

  # Optional macOS DMG creation  
  build-dmg:
    if: ${{ vars.JDEPLOY_CREATE_DMG == 'true' }}
    runs-on: macos-latest
    
    steps:
    - uses: actions/checkout@v4
    
    - name: Set up JDK
      uses: actions/setup-java@v4
      with:
        java-version: '21'
        distribution: 'temurin'
    
    - name: Make gradlew executable
      run: chmod +x ./gradlew
    
    - name: Build with Gradle
      run: ./gradlew build
    
    - name: Build DMG
      uses: shannah/jdeploy-action-dmg@main
      with:
        github_token: ${{ secrets.GITHUB_TOKEN }}
        developer_id_application: ${{ secrets.DEVELOPER_ID_APPLICATION }}
        developer_id_application_password: ${{ secrets.DEVELOPER_ID_APPLICATION_PASSWORD }}
        apple_id: ${{ secrets.APPLE_ID }}
        apple_id_password: ${{ secrets.APPLE_ID_PASSWORD }}
        apple_team_id: ${{ secrets.APPLE_TEAM_ID }}
```

### Workflow Features:

- **Automatic Triggers**: Runs on pushes to all branches (except gh-pages) and tags
- **Cross-Platform Bundles**: Creates installers for Windows, macOS, and Linux
- **Optional DMG**: macOS-specific bundle creation when `JDEPLOY_CREATE_DMG` variable is set
- **GitHub Releases**: Automatically uploads artifacts to GitHub releases for tagged builds

### Repository Variables (Optional):
- Set `JDEPLOY_CREATE_DMG` to `'true'` in repository settings to enable macOS DMG creation

### Secrets for macOS Code Signing (Optional):
If creating signed macOS DMG bundles, add these secrets to your repository:
- `DEVELOPER_ID_APPLICATION`
- `DEVELOPER_ID_APPLICATION_PASSWORD`  
- `APPLE_ID`
- `APPLE_ID_PASSWORD`
- `APPLE_TEAM_ID`

## Verification Checklist

- [ ] Java project builds successfully
- [ ] JAR file is executable with `java -jar`
- [ ] `package.json` exists with correct jDeploy configuration
- [ ] JAR path in package.json matches actual build output
- [ ] Java version specified matches project requirements
- [ ] For JavaFX apps: `javafx: true` is set

## Next Steps

Once setup is complete, you can use jDeploy to:

1. **Package for testing:** `jdeploy package`
2. **Create native bundles:** `jdeploy bundle` 
3. **Publish to NPM:** `jdeploy publish`

## Common Issues

- **JAR not found**: Verify the `jar` path in package.json matches your build output location
- **Main class not found**: Ensure your JAR's MANIFEST.MF includes the Main-Class attribute
- **Missing dependencies**: For non-shaded JARs, ensure the lib/ directory is created during build
- **JavaFX runtime issues**: Set `"javafx": true` and verify JavaFX modules are included in your build