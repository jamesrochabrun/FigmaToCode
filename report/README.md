# FigmaToCode - Comprehensive Analysis Report

This report provides an in-depth analysis of the FigmaToCode plugin, covering its architecture, capabilities, limitations, and technical implementation.

## Report Contents

### 📋 Overview Documents
- **[Architecture](./architecture.md)** - System architecture, monorepo structure, and component organization
- **[Capabilities](./capabilities.md)** - Complete feature set and plugin capabilities
- **[Limitations](./limitations.md)** - Known constraints and unsupported features
- **[Platform Support](./platform-support.md)** - Platform compatibility and web-specific features

### 🔧 Technical Details
- **[Conversion Pipeline](./conversion-pipeline.md)** - Detailed explanation of the Figma-to-code conversion process
- **[Frameworks](./frameworks.md)** - Supported frameworks and their implementations
- **[Algorithms](./algorithms.md)** - Key algorithms for layout detection, color conversion, and optimization
- **[Technology Stack](./technology-stack.md)** - All technologies, libraries, and tools used

### 👨‍💻 Developer Resources
- **[Getting Started](./getting-started.md)** - Guide for developers who want to contribute or understand the codebase
- **[Plugin Settings](./plugin-settings.md)** - All available settings and configuration options

## Quick Summary

**FigmaToCode** is a production-grade Figma plugin that converts design files into code for multiple frameworks including HTML, React, Tailwind, Flutter, SwiftUI, and Jetpack Compose.

### Key Metrics
- **Supported Frameworks**: 5 main frameworks (8 output modes)
- **Codebase Size**: ~7,786 lines of backend code
- **Architecture**: Monorepo with TypeScript, built with Turborepo and esbuild
- **Plugin Type**: Figma plugin with Dev Mode integration

### Primary Use Cases
1. Converting Figma designs to responsive layouts
2. Extracting color palettes and gradients
3. Generating text style definitions
4. Creating component code from design components
5. Rapid prototyping and handoff to developers

### Target Platforms
- **Web**: HTML, React (JSX), Svelte, Tailwind, styled-components
- **Mobile**: Flutter (iOS/Android), SwiftUI (iOS), Jetpack Compose (Android)
- **Not Web-Specific**: Supports both web and mobile frameworks

## Report Metadata

- **Analysis Date**: 2025-11-12
- **Repository**: [bernaferrari/FigmaToCode](https://github.com/bernaferrari/FigmaToCode)
- **Latest Commit Analyzed**: 427381b
- **Plugin ID**: 842128343887142055
- **Package Manager**: pnpm@9.14.4

---

*This report was generated through comprehensive codebase analysis, including exploration of all major components, builders, algorithms, and utilities.*
