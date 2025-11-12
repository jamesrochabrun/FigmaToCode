# Capabilities

This document outlines all the features and capabilities of the FigmaToCode plugin.

## Table of Contents
- [Core Features](#core-features)
- [Conversion Capabilities](#conversion-capabilities)
- [Output Formats](#output-formats)
- [Design System Support](#design-system-support)
- [Advanced Features](#advanced-features)
- [User Interface Features](#user-interface-features)
- [Developer Features](#developer-features)

## Core Features

### 1. Multi-Framework Code Generation

FigmaToCode can convert designs into code for multiple frameworks:

| Framework | Variants | Status |
|-----------|----------|--------|
| **HTML** | Pure HTML, React JSX, Svelte, styled-components | ✅ Full Support |
| **Tailwind CSS** | HTML + Tailwind, JSX + Tailwind | ✅ Full Support |
| **Flutter** | Snippet, Stateless Widget, Full App | ✅ Full Support |
| **SwiftUI** | Snippet, Struct, Preview | ✅ Full Support |
| **Jetpack Compose** | Snippet, Composable, Screen | ✅ Full Support |

### 2. Real-Time Conversion

- **Auto-conversion on selection**: Code updates automatically when you select nodes
- **Debounced processing**: Prevents excessive processing during rapid selection changes
- **Loading states**: Visual feedback during async operations
- **Incremental updates**: Only re-processes when necessary

### 3. Responsive Layout Detection

Automatically converts Figma's auto-layout to framework-specific responsive layouts:

- **HTML/Tailwind**: Flexbox and Grid
- **Flutter**: Column, Row, Stack, Wrap
- **SwiftUI**: HStack, VStack, ZStack
- **Compose**: Column, Row, Box

**Detected properties**:
- Layout direction (horizontal/vertical)
- Alignment and justification
- Spacing and gaps
- Wrapping behavior
- Sizing constraints (fill, hug, fixed)

### 4. Live Preview

Built-in HTML preview with:
- **View modes**: Desktop, mobile, precision (single-element)
- **Background toggle**: White or black background
- **Expandable panel**: Resize preview area
- **Real-time updates**: Preview updates with code changes

## Conversion Capabilities

### Layout Conversion

#### Auto-Layout
- ✅ Horizontal and vertical layouts
- ✅ Spacing and padding
- ✅ Alignment properties
- ✅ Sizing constraints (FILL, HUG, FIXED)
- ✅ Wrapping (with framework support)
- ✅ Absolute positioning within auto-layout

#### Manual Positioning
- ✅ Absolute positioning
- ✅ Relative positioning
- ✅ Z-index ordering
- ✅ Rotation (with CSS transforms)
- ⚠️ Constraints (simplified to common patterns)

### Visual Properties

#### Colors
- ✅ Solid fills
- ✅ Gradients (linear, radial, angular)
- ✅ Multiple fills
- ✅ Color variables (design tokens)
- ✅ Opacity
- ✅ Hex, RGB, RGBA conversion
- ✅ Tailwind color matching

#### Borders
- ✅ Solid borders
- ✅ Border radius (including individual corners)
- ✅ Border width
- ✅ Border color
- ✅ Gradient borders (where supported)

#### Effects
- ✅ Drop shadows
- ✅ Inner shadows
- ✅ Multiple shadows
- ✅ Layer blur
- ✅ Background blur (backdrop-filter)
- ✅ Blend modes

#### Typography
- ✅ Font family, weight, size
- ✅ Line height
- ✅ Letter spacing
- ✅ Text alignment
- ✅ Text decoration (underline, strikethrough)
- ✅ Text transform (uppercase, lowercase, capitalize)
- ✅ Rich text with multiple styles
- ✅ Text color and gradients
- ✅ Truncation support

### Advanced Conversions

#### Vector Graphics
- ✅ **SVG embedding**: Embed vectors as inline SVG
- ✅ **Icon detection**: Automatic detection of icon-like vectors
- ✅ **Color variable preservation**: SVG colors mapped to CSS variables
- ✅ **Optimization**: Clean SVG output
- ⚠️ **Framework support**: HTML/Tailwind only (Flutter/SwiftUI use placeholders)

#### Images
- ✅ **Placeholder generation**: Image fills replaced with placeholders
- ✅ **Base64 inline**: Optional base64 embedding (HTML only)
- ✅ **Aspect ratio preservation**: Maintains image proportions
- ⚠️ **Asset management**: No automatic asset pipeline

#### Components
- ✅ Component instances recognized
- ✅ Component properties preserved where possible
- ✅ Nested components supported
- ⚠️ Variants have limited support

## Output Formats

### HTML Modes

#### 1. Pure HTML (`html`)
```html
<div style="display: flex; flex-direction: column; gap: 16px;">
  <div style="background-color: #3B82F6;">...</div>
</div>
```

**Features**:
- Inline styles or external CSS (configurable)
- Semantic HTML when possible
- Class-based or inline styling

#### 2. React JSX (`html_jsx`)
```jsx
<div style={{display: 'flex', flexDirection: 'column', gap: '16px'}}>
  <div style={{backgroundColor: '#3B82F6'}}>...</div>
</div>
```

**Features**:
- CamelCase properties
- JSX-compatible syntax
- React component structure

#### 3. Svelte (`html_svelte`)
```svelte
<div class="container">...</div>

<style>
.container {
  display: flex;
  flex-direction: column;
}
</style>
```

**Features**:
- Scoped styles
- Svelte component structure
- Class-based styling

#### 4. styled-components (`html_styled_components`)
```jsx
const Container = styled.div`
  display: flex;
  flex-direction: column;
  gap: 16px;
`;
```

**Features**:
- CSS-in-JS
- Tagged template literals
- TypeScript-compatible

### Tailwind Modes

#### 1. Tailwind HTML (`tailwind`)
```html
<div class="flex flex-col gap-4 bg-blue-500 rounded-lg p-4">
  ...
</div>
```

#### 2. Tailwind JSX (`tailwind_jsx`)
```jsx
<div className="flex flex-col gap-4 bg-blue-500 rounded-lg p-4">
  ...
</div>
```

**Features**:
- Utility-first classes
- Arbitrary values for custom properties `[value]`
- Color matching to Tailwind palette
- Custom prefix support
- Tailwind v4 support

### Flutter Modes

#### 1. Snippet
```dart
Container(
  padding: EdgeInsets.all(16),
  child: Column(children: [...])
)
```

#### 2. Stateless Widget
```dart
class MyWidget extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Container(...);
  }
}
```

#### 3. Full App
```dart
import 'package:flutter/material.dart';

void main() => runApp(MyApp());

class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(home: MyWidget());
  }
}
```

### SwiftUI Modes

#### 1. Snippet
```swift
VStack(spacing: 16) {
  Text("Hello")
}
```

#### 2. Struct
```swift
struct MyView: View {
  var body: some View {
    VStack(spacing: 16) {
      Text("Hello")
    }
  }
}
```

#### 3. Preview
```swift
struct MyView_Previews: PreviewProvider {
  static var previews: some View {
    MyView()
  }
}
```

### Compose Modes

#### 1. Snippet
```kotlin
Column(modifier = Modifier.padding(16.dp)) {
  Text("Hello")
}
```

#### 2. Composable
```kotlin
@Composable
fun MyScreen() {
  Column(...) { ... }
}
```

#### 3. Screen
Full screen with Scaffold structure

## Design System Support

### Color Variables

- ✅ **Detection**: Automatically detects bound color variables
- ✅ **Naming**: Uses variable names in generated code
- ✅ **Fallbacks**: Provides fallback values
- ✅ **CSS Variables**: Generates `var(--color-name, #fallback)`
- ✅ **SVG integration**: Replaces colors in embedded SVGs

Example output:
```css
color: var(--primary-500, #3B82F6);
```

### Text Styles

- ✅ **Extraction**: Pulls all text styles from selection
- ✅ **Separate export**: Available in dedicated "Text Styles" tab
- ✅ **Framework-specific**: Generates appropriate code for each framework

### Color Palette Extraction

- ✅ **Auto-extraction**: Finds all colors in selection
- ✅ **Contrast ratios**: Calculates WCAG contrast ratios
- ✅ **Tailwind mapping**: Matches to closest Tailwind colors
- ✅ **Gradient export**: Exports gradients with framework syntax
- ✅ **Click to copy**: One-click copying of individual colors

## Advanced Features

### 1. Icon Detection Algorithm

Automatically detects icon-like elements based on:
- Node type (vector, boolean operations, primitives)
- Size constraints (max 64x64px for primitives)
- SVG export settings
- Container analysis
- Content validation

**Result**: Enables SVG embedding for detected icons

### 2. Rotation Handling

Correctly handles rotated elements:
- Calculates cumulative rotation from parent hierarchy
- Applies CSS transforms
- Adjusts bounding box calculations
- Preserves visual accuracy

### 3. Smart Layout Detection

Analyzes layout patterns and generates optimal code:
- Auto-layout → Flexbox/Grid/Stack
- Negative spacing handling
- Gap vs margin optimization
- Constraint simplification

### 4. Color Matching

- **Nearest color**: Finds closest Tailwind color
- **Threshold-based**: Only rounds within configurable threshold (default 15%)
- **Arbitrary values**: Uses `[#hex]` for exact colors
- **Gradient support**: Converts to framework-specific gradient syntax

### 5. Performance Tracking

Built-in performance monitoring:
- Tracks API call counts
- Measures processing time
- Identifies bottlenecks
- Optimizes frequently-called operations

## User Interface Features

### Code View

- ✅ **Syntax highlighting**: Language-aware highlighting
- ✅ **Copy to clipboard**: One-click code copying
- ✅ **Auto-scroll**: Scrolls to show new content
- ✅ **Line numbers**: Optional line numbering
- ✅ **Word wrap**: Configurable word wrapping

### Preview Panel

- ✅ **Live rendering**: Real-time HTML preview
- ✅ **Responsive views**: Desktop, mobile, precision modes
- ✅ **Background toggle**: Switch between white/black backgrounds
- ✅ **Resizable**: Drag to resize preview area
- ✅ **Expandable**: Full-screen preview mode

### Settings Panel

Comprehensive configuration options:
- Framework selection
- Output mode
- Layer name visibility
- Color variable usage
- Vector embedding
- Image embedding
- Tailwind rounding
- Custom prefix
- Base font size
- Base font family
- Threshold percentage

### Warnings System

- ✅ **Real-time warnings**: Shows unsupported features
- ✅ **Detailed messages**: Explains what's not supported
- ✅ **Dismissible**: Can hide warnings
- ✅ **Warning counts**: Badge showing warning count

### Multi-Tab Interface

- **Code tab**: Generated code output
- **Text Styles tab**: Extracted text styles
- **Colors tab**: Color palette and gradients
- **Settings tab**: Configuration options

## Developer Features

### Dev Mode Integration

Integrated with Figma's Dev Mode:
- ✅ **Right-click menu**: "Copy as..." options
- ✅ **Multiple languages**: All supported frameworks in menu
- ✅ **Quick access**: No need to open plugin UI
- ✅ **Context-aware**: Only shows relevant options

### Debug Mode

Development-only features:
- ✅ **Debug UI**: Next.js app at `http://localhost:3000`
- ✅ **JSON export**: View raw Figma JSON
- ✅ **Comparison view**: Old vs new conversion
- ✅ **Component browser**: Browse all UI components
- ✅ **Test cases**: Preset test scenarios

### Plugin API

Exposed functionality:
- `convertToCode(nodes, settings)` - Main conversion
- `nodesToJSON(nodes)` - Export to JSON
- `generateHTMLPreview(nodes)` - Generate preview
- Framework-specific `*Main` functions

### Extensibility

- ✅ **Plugin architecture**: Easy to extend
- ✅ **Builder pattern**: Consistent API for new frameworks
- ✅ **Shared utilities**: Reusable common functions
- ✅ **Type-safe**: Full TypeScript support

## Configuration Options

### Plugin Settings

All available settings (from `defaultPluginSettings`):

```typescript
{
  framework: "HTML" | "Tailwind" | "Flutter" | "SwiftUI" | "Compose",
  showLayerNames: boolean,           // Show Figma layer names in code
  useColorVariables: boolean,        // Use design tokens
  embedImages: boolean,              // Inline images as base64
  embedVectors: boolean,             // Embed SVGs inline
  roundTailwindValues: boolean,      // Round to Tailwind scale
  roundTailwindColors: boolean,      // Match Tailwind color palette
  customTailwindPrefix: string,      // Custom Tailwind prefix
  baseFontSize: number,              // Base font size for rem
  useTailwind4: boolean,             // Use Tailwind v4 syntax
  thresholdPercent: number,          // Rounding threshold (default 15%)
  baseFontFamily: string,            // Base font family

  // Framework-specific generation modes
  flutterGenerationMode: "snippet" | "stateless" | "fullApp",
  swiftUIGenerationMode: "snippet" | "struct" | "preview",
  composeGenerationMode: "snippet" | "composable" | "screen",
  htmlGenerationMode: "html" | "jsx" | "svelte" | "styled-components"
}
```

### Per-Framework Options

#### Tailwind Options
- Custom prefix (e.g., `tw-`)
- Tailwind v4 mode
- Color rounding
- Value rounding
- Threshold percentage

#### HTML Options
- Generation mode (html/jsx/svelte/styled-components)
- Inline vs external styles
- Class naming strategy

#### Flutter Options
- Generation mode (snippet/stateless/fullApp)
- Material version
- Widget naming

#### SwiftUI Options
- Generation mode (snippet/struct/preview)
- Preview device

#### Compose Options
- Generation mode (snippet/composable/screen)
- Material 3 support

## Supported Node Types

| Node Type | Support Level | Notes |
|-----------|---------------|-------|
| **FRAME** | ✅ Full | Primary container |
| **GROUP** | ✅ Full | Inlined to parent |
| **RECTANGLE** | ✅ Full | Div/Container/Box |
| **TEXT** | ✅ Full | Rich text support |
| **ELLIPSE** | ✅ Full | Border-radius or Circle |
| **VECTOR** | ⚠️ Partial | SVG embedding only |
| **BOOLEAN_OPERATION** | ⚠️ Partial | SVG embedding only |
| **COMPONENT** | ✅ Full | Treated as group |
| **INSTANCE** | ✅ Full | Treated as group |
| **LINE** | ⚠️ Limited | Basic support |
| **POLYGON** | ⚠️ Limited | Basic support |
| **STAR** | ⚠️ Limited | Basic support |
| **SLICE** | ❌ None | Ignored |

## Performance Characteristics

### Fast Operations
- Single node conversion: <100ms
- Small selections (1-10 nodes): <500ms
- Auto-layout detection: <50ms
- Color extraction: <100ms

### Slower Operations
- Large selections (50+ nodes): 2-5 seconds
- SVG export: 200-500ms per vector
- Complex text with many segments: 100-300ms
- Deep nesting (10+ levels): Slower due to recursion

### Optimization Strategies
- Debounced selection changes (300ms)
- Parallel async operations
- Incremental processing
- Memoization of variable lookups

---

FigmaToCode provides a comprehensive set of capabilities for converting designs to production-ready code across multiple frameworks, with extensive configuration options and intelligent feature detection.
