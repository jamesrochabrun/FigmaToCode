# Architecture

This document details the architectural design and structure of the FigmaToCode plugin.

## Table of Contents
- [Overview](#overview)
- [Monorepo Structure](#monorepo-structure)
- [Backend Architecture](#backend-architecture)
- [Plugin Architecture](#plugin-architecture)
- [Design Patterns](#design-patterns)
- [Data Flow](#data-flow)

## Overview

FigmaToCode uses a **monorepo architecture** with clear separation between conversion logic (backend), user interface (plugin-ui), and application bundles (apps). The architecture is designed for:

- **Extensibility**: Easy to add new frameworks
- **Maintainability**: Shared code and utilities
- **Performance**: Fast incremental builds with esbuild
- **Type Safety**: Full TypeScript coverage

## Monorepo Structure

```
FigmaToCode/
├── packages/                    # Shared packages
│   ├── backend/                # Core conversion engine
│   ├── plugin-ui/              # Shared UI components
│   ├── types/                  # TypeScript type definitions
│   ├── eslint-config-custom/   # ESLint configuration
│   └── tsconfig/               # TypeScript configurations
├── apps/                       # Application bundles
│   ├── plugin/                 # Main Figma plugin
│   │   ├── plugin-src/         # Backend entry (compiles to code.js)
│   │   └── ui-src/             # UI entry (compiles to index.html)
│   └── debug/                  # Development debug UI (Next.js)
└── assets/                     # Images and documentation assets
```

### Package Responsibilities

#### `packages/backend`
The heart of the conversion logic. Contains:
- Framework-specific code generators
- Shared utilities and algorithms
- AltNode intermediate representation
- JSON node processing

**Size**: ~7,786 lines of core conversion code

#### `packages/plugin-ui`
Reusable UI components for the plugin interface:
- React components (buttons, inputs, modals)
- Syntax highlighting
- Preview panel
- Settings interface

#### `packages/types`
Centralized TypeScript type definitions:
- Figma API type extensions
- AltNode types
- Plugin settings types
- Framework-specific types

#### `apps/plugin`
The actual Figma plugin assembly:
- **plugin-src**: Runs in Figma's plugin sandbox (Node-like environment)
- **ui-src**: Runs in iframe (browser environment)
- Bridges backend and UI packages

#### `apps/debug`
Development-only Next.js application:
- Browse conversion examples
- Debug JSON output
- Test UI components in isolation
- Accessible at `http://localhost:3000` during development

## Backend Architecture

The backend follows a **modular, framework-oriented** architecture:

```
packages/backend/src/
├── altNodes/               # Intermediate representation
│   ├── altConversion/     # Figma node → AltNode conversion
│   └── altMixins.ts       # AltNode utility functions
├── common/                 # Shared utilities
│   ├── commonPosition.ts  # Position and size calculations
│   ├── commonColor.ts     # Color conversion utilities
│   ├── commonPadding.ts   # Padding/spacing calculations
│   ├── indentString.ts    # Code formatting
│   └── ...
├── html/                   # HTML/React/Svelte generators
│   ├── htmlMain.ts        # Entry point
│   ├── htmlDefaultBuilder.ts
│   ├── htmlTextBuilder.ts
│   └── builderImpl/       # Property-specific builders
├── tailwind/               # Tailwind CSS generator
│   ├── tailwindMain.ts
│   ├── tailwindDefaultBuilder.ts
│   ├── tailwindTextBuilder.ts
│   └── builderImpl/
├── flutter/                # Flutter/Dart generator
│   ├── flutterMain.ts
│   ├── flutterDefaultBuilder.ts
│   ├── flutterTextBuilder.ts
│   └── builderImpl/
├── swiftui/                # SwiftUI generator
│   └── ... (same structure)
├── compose/                # Jetpack Compose generator
│   └── ... (same structure)
└── code.ts                 # Main entry point
```

### Framework Module Pattern

Each framework follows a consistent structure:

```
{framework}/
├── {framework}Main.ts              # Entry point and tree traversal
├── {framework}DefaultBuilder.ts    # General element builder
├── {framework}TextBuilder.ts       # Text element builder
└── builderImpl/                    # Property implementations
    ├── {framework}Color.ts         # Color/gradient handling
    ├── {framework}Size.ts          # Width/height/sizing
    ├── {framework}Border.ts        # Border/stroke handling
    ├── {framework}Padding.ts       # Padding/margin
    ├── {framework}Position.ts      # Positioning/layout
    ├── {framework}Blend.ts         # Blend modes/opacity
    ├── {framework}Shadow.ts        # Shadow effects
    └── {framework}AutoLayout.ts    # Auto-layout conversion
```

This consistency makes it easy to:
1. Add new frameworks by copying the pattern
2. Understand any framework's implementation
3. Share common logic across frameworks

## Plugin Architecture

The plugin uses Figma's **two-thread architecture**:

### Main Thread (Sandbox)
Runs `code.js` in a restricted Node-like environment:
- Has access to Figma API (`figma.*`)
- No DOM access
- No network access (plugin uses `"allowedDomains": ["none"]`)
- Performs all node traversal and conversion

**Key file**: `apps/plugin/plugin-src/code.ts`

### UI Thread (iframe)
Runs `index.html` in a browser environment:
- Full DOM and React access
- No direct Figma API access
- Renders UI, handles user interaction
- Communicates with main thread via `postMessage`

**Key file**: `apps/plugin/ui-src/App.tsx`

### Communication Protocol

```typescript
// Main → UI
figma.ui.postMessage({
  type: "codeUpdate",
  code: "...",
  warnings: [...],
  colors: [...]
});

// UI → Main
parent.postMessage({
  pluginMessage: {
    type: "convert",
    settings: {...}
  }
}, "*");
```

Message types:
- `codeUpdate` - Send generated code to UI
- `selectionChange` - Notify UI of new selection
- `settingsUpdate` - Update conversion settings
- `export` - Trigger export operation
- `copyCode` - Copy code to clipboard

## Design Patterns

### 1. Builder Pattern

Every framework uses the builder pattern for code generation:

```typescript
class HtmlDefaultBuilder {
  private styles: string[] = [];
  private attributes: string[] = [];

  commonPositionStyles(): this {
    // Add position-related styles
    return this;
  }

  commonShapeStyles(): this {
    // Add shape-related styles
    return this;
  }

  build(additionalStyles?: string): string {
    // Combine and return final code
  }
}

// Usage
const code = new HtmlDefaultBuilder(node, settings)
  .commonPositionStyles()
  .commonShapeStyles()
  .build();
```

**Benefits**:
- Method chaining for fluent API
- Incremental property accumulation
- Easy to extend with new properties
- Clear separation of concerns

### 2. Strategy Pattern

Different frameworks implement the same conversion interface:

```typescript
// Common interface (implicit)
function {framework}Main(
  node: AltSceneNode,
  settings: PluginSettings
): string

// Different implementations
htmlMain(node, settings)      // → HTML string
tailwindMain(node, settings)  // → Tailwind string
flutterMain(node, settings)   // → Dart string
```

### 3. Intermediate Representation Pattern

Figma nodes → **AltNodes** → Framework code

This decouples Figma's API from code generation:
- **AltNodes** are framework-agnostic
- Can be transformed without affecting original design
- Enable optimizations before code generation
- Allow for future Figma API changes without breaking generators

### 4. Visitor Pattern (Tree Traversal)

Each framework recursively visits the node tree:

```typescript
function htmlMain(node: AltSceneNode): string {
  if (node.children && node.children.length > 0) {
    const childrenCode = node.children
      .map(child => htmlMain(child))  // Recursive visit
      .join("\n");
    return wrapInContainer(node, childrenCode);
  }
  return generateLeafNode(node);
}
```

### 5. Factory Pattern

Different builders created based on node type:

```typescript
function getBuilder(node: AltSceneNode) {
  if (node.type === "TEXT") {
    return new HtmlTextBuilder(node, settings);
  }
  return new HtmlDefaultBuilder(node, settings);
}
```

## Data Flow

### Complete Conversion Pipeline

```
┌─────────────────────────────────────────────────────────────┐
│ 1. User Selection in Figma                                  │
│    figma.currentPage.selection                              │
└────────────────┬────────────────────────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────────────────────────┐
│ 2. Export to JSON REST v1                                   │
│    node.exportAsync({ format: "JSON_REST_V1" })             │
└────────────────┬────────────────────────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────────────────────────┐
│ 3. Process Node Pairs (Recursive)                           │
│    • Add parent references                                  │
│    • Calculate rotation                                     │
│    • Inline groups                                          │
│    • Generate unique names                                  │
│    • Extract text segments                                  │
│    • Process color variables                                │
│    • Detect icons                                           │
│    • Export SVGs (if vectorizable)                          │
└────────────────┬────────────────────────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────────────────────────┐
│ 4. AltNode Intermediate Representation                      │
│    Enhanced nodes with:                                     │
│    • styledTextSegments                                     │
│    • cumulativeRotation                                     │
│    • uniqueName                                             │
│    • canBeFlattened                                         │
│    • svg (embedded)                                         │
└────────────────┬────────────────────────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────────────────────────┐
│ 5. Framework-Specific Code Generation                       │
│    Route to: htmlMain | tailwindMain | flutterMain | ...    │
└────────────────┬────────────────────────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────────────────────────┐
│ 6. Builder Execution (Recursive Tree Traversal)             │
│    For each node:                                           │
│    • Create builder (Default or Text)                       │
│    • Apply common styles                                    │
│    • Process children recursively                           │
│    • Build final code string                                │
└────────────────┬────────────────────────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────────────────────────┐
│ 7. Post-Processing                                          │
│    • Indentation                                            │
│    • Formatting                                             │
│    • Add imports/wrappers (if needed)                       │
└────────────────┬────────────────────────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────────────────────────┐
│ 8. Send to UI                                               │
│    figma.ui.postMessage({ type: "codeUpdate", ... })        │
└────────────────┬────────────────────────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────────────────────────┐
│ 9. Display in Plugin UI                                     │
│    • Syntax highlighting                                    │
│    • Copy to clipboard                                      │
│    • Live preview                                           │
└─────────────────────────────────────────────────────────────┘
```

### Parallel Operations

The plugin performs several operations in parallel for performance:

```typescript
await Promise.all([
  processColorVariables(node),
  renderAndAttachSVG(node),
  getStyledTextSegments(node)
]);
```

### State Management

**Plugin Settings**:
- Stored in `figma.clientStorage`
- Persisted across sessions
- Synced between main and UI threads

**Selection State**:
- Managed by Figma
- Monitored with `figma.on("selectionchange")`
- Debounced to prevent excessive processing

## Performance Considerations

### Optimizations

1. **Incremental Builds**: esbuild only recompiles changed files
2. **Lazy Processing**: Only process selected nodes, not entire page
3. **Parallel Export**: SVG and color variable processing in parallel
4. **Memoization**: Cache variable names and color mappings
5. **Debouncing**: Delay conversion on selection change (prevents rapid updates)

### Performance Counters

The codebase includes built-in performance tracking:

```typescript
let counterGetNodeByIdAsync = 0;
let counterGetStyledTextSegments = 0;
let counterProcessColorVariables = 0;
```

These help identify bottlenecks during development.

### Known Bottlenecks

1. **Large selections** (>100 nodes): Exponential processing time
2. **SVG export**: Async and can be slow for complex vectors
3. **Deep nesting**: Recursive traversal overhead
4. **Text segments**: Rich text processing is expensive

## Extensibility

### Adding a New Framework

To add support for a new framework:

1. Create directory: `packages/backend/src/{framework}/`
2. Implement main function: `{framework}Main.ts`
3. Create builders: `{framework}DefaultBuilder.ts`, `{framework}TextBuilder.ts`
4. Implement property handlers in `builderImpl/`
5. Add to routing in `convertToCode.ts`
6. Add manifest entry in `manifest.json`
7. Add UI option in plugin settings

Example minimal implementation:

```typescript
// packages/backend/src/vue/vueMain.ts
export function vueMain(
  sceneNode: AltSceneNode[],
  settings: PluginSettings
): string {
  const builder = new VueDefaultBuilder(sceneNode[0], settings);
  return builder
    .commonPositionStyles()
    .commonShapeStyles()
    .build();
}
```

### Plugin Extension Points

- **Custom settings**: Add to `PluginSettings` type
- **Custom builders**: Extend base builder classes
- **Custom utilities**: Add to `common/` directory
- **UI components**: Add to `plugin-ui/src/components/`

## Build System

### Turborepo Configuration

```json
{
  "pipeline": {
    "build": {
      "dependsOn": ["^build"],
      "outputs": ["dist/**"]
    },
    "dev": {
      "cache": false,
      "persistent": true
    }
  }
}
```

**Key features**:
- Parallel builds across packages
- Dependency-aware build order
- Caching for faster rebuilds
- Concurrent development mode

### esbuild Configuration

Fast bundling with:
- TypeScript compilation
- Tree shaking
- Minification
- Source maps (in dev mode)

**Build times**: Typically <1 second for incremental changes

---

This architecture enables FigmaToCode to be maintainable, performant, and extensible while handling complex design-to-code conversions across multiple frameworks.
