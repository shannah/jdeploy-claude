# jDeploy Claude Reference Repository

This repository serves as a comprehensive reference for Claude to help Java developers set up and configure jDeploy for their projects.

## What is jDeploy?

jDeploy is a tool that allows you to deploy Java applications as native applications or NPM packages without requiring Java to be pre-installed on target machines. It handles the bundling of your Java application with the appropriate JRE.

## Repository Structure

- **`docs/`** - Comprehensive documentation and guides
- **`templates/`** - Project templates for different Java application types
- **`scripts/`** - Utility scripts for setup and validation
- **`.jdeploy/`** - Example jDeploy configurations
- **`CLAUDE.md`** - Specific instructions for Claude AI assistant

## Quick Start for Developers

1. Ensure your Java project builds successfully
2. Run `jdeploy init` in your project root
3. Configure your `package.json` with jDeploy settings
4. Run `jdeploy publish` to deploy

## Supported Project Types

This reference includes templates and examples for:

- **JavaFX Applications** - Desktop apps with modern UI
- **Swing Applications** - Traditional desktop applications  
- **Spring Boot + Swing** - Enterprise desktop apps with Spring
- **FXGL Games** - Java-based game development
- **Modular JavaFX** - Projects using Java module system
- **Codename One** - Cross-platform mobile applications

## For Claude AI

See `CLAUDE.md` for specific instructions on how to help developers with jDeploy setup, configuration, and troubleshooting.