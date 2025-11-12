# Plugin Settings

This document provides a comprehensive reference for all available plugin settings and configuration options.

## Table of Contents
- [Overview](#overview)
- [Framework Selection](#framework-selection)
- [General Settings](#general-settings)
- [Tailwind-Specific Settings](#tailwind-specific-settings)
- [Framework Generation Modes](#framework-generation-modes)
- [Settings API](#settings-api)
- [Default Values](#default-values)

## Overview

FigmaToCode provides extensive configuration options to customize code generation behavior. Settings are persisted locally using `figma.clientStorage` and synchronized between the plugin backend and UI.

## Framework Selection

### Available Frameworks

```typescript
type Framework =
  | "HTML"
  | "Tailwind"
  | "Flutter"
  | "SwiftUI"
  | "Compose";
```

**Default**: `"HTML"`

### Framework Descriptions

| Framework | Description | Best For |
|-----------|-------------|----------|
| **HTML** | Pure HTML with CSS | Web development, learning |
| **Tailwind** | Tailwind CSS utility classes | Modern web development |
| **Flutter** | Dart widgets | Cross-platform mobile apps |
| **SwiftUI** | Apple's declarative UI | iOS/macOS apps |
| **Compose** | Jetpack Compose | Modern Android apps |

## General Settings

### Show Layer Names

**Property**: `showLayerNames`
**Type**: `boolean`
**Default**: `true`

**Description**: Include Figma layer names as comments or identifiers in generated code.

**Example**:

**Enabled** (`true`):
```html
<!-- Button Container -->
<div style="...">
  <!-- Label -->
  <div style="...">Click me</div>
</div>
```

**Disabled** (`false`):
```html
<div style="...">
  <div style="...">Click me</div>
</div>
```

**Use Cases**:
- Enable: Better code readability, easier to identify elements
- Disable: Cleaner code, no comments

---

### Use Color Variables

**Property**: `useColorVariables`
**Type**: `boolean`
**Default**: `true`

**Description**: Replace color values with CSS variable references when colors are bound to Figma variables.

**Example**:

**Enabled** (`true`):
```css
color: var(--primary-500, #3B82F6);
background-color: var(--neutral-100, #F3F4F6);
```

**Disabled** (`false`):
```css
color: #3B82F6;
background-color: #F3F4F6;
```

**Benefits**:
- Maintains design system color tokens
- Enables theming
- Provides fallback values

---

### Embed Images

**Property**: `embedImages`
**Type**: `boolean`
**Default**: `false`

**Description**: Embed images as base64-encoded data URLs instead of placeholders.

**Frameworks**: HTML, Tailwind only (limited support in others)

**Example**:

**Enabled** (`true`):
```html
<img src="data:image/png;base64,iVBORw0KGgoAAAANSUhEUgA..." />
```

**Disabled** (`false`):
```html
<img src="https://via.placeholder.com/400x300" />
```

**Considerations**:
- ⚠️ **File size**: Base64 encoding increases size by ~33%
- ⚠️ **Performance**: Large images can slow down page load
- ✅ **Self-contained**: No external dependencies
- ✅ **Quick prototyping**: Works immediately

**Recommendation**: Disable for production, enable for quick prototypes

---

### Embed Vectors

**Property**: `embedVectors`
**Type**: `boolean`
**Default**: `true`

**Description**: Embed vector graphics as inline SVG instead of placeholders.

**Frameworks**: HTML, Tailwind only

**Example**:

**Enabled** (`true`):
```html
<svg width="24" height="24" viewBox="0 0 24 24">
  <path d="M12 2L2 7v10c0 5.5 3.8 10.7 10 12 6.2-1.3 10-6.5 10-12V7l-10-5z" fill="currentColor"/>
</svg>
```

**Disabled** (`false`):
```html
<!-- Vector: icon-shield -->
<div style="width: 24px; height: 24px; background-color: #000;"></div>
```

**Benefits**:
- ✅ Scalable graphics
- ✅ Color variable support in SVG
- ✅ No external files
- ✅ Better than placeholder

**Considerations**:
- ⚠️ Complex vectors generate verbose code
- ⚠️ Not supported in Flutter/SwiftUI/Compose

---

## Tailwind-Specific Settings

### Round Tailwind Values

**Property**: `roundTailwindValues`
**Type**: `boolean`
**Default**: `true`

**Description**: Round size values to Tailwind's spacing scale instead of using arbitrary values.

**Example**:

**Enabled** (`true`):
```html
<!-- 18px → rounds to 16px (w-4) -->
<div class="w-4 h-8 p-6">
```

**Disabled** (`false`):
```html
<!-- Uses exact values -->
<div class="w-[18px] h-[32px] p-[24px]">
```

**Tailwind Scale**: 0, 1, 2, 4, 6, 8, 12, 16, 20, 24, 32, 40, 48, 64, 80, 96, 128, 256...

**Algorithm**: Uses `nearestValueWithThreshold()` with configurable threshold

---

### Round Tailwind Colors

**Property**: `roundTailwindColors`
**Type**: `boolean`
**Default**: `true`

**Description**: Match colors to Tailwind's color palette instead of using arbitrary hex values.

**Example**:

**Enabled** (`true`):
```html
<!-- #3B82F6 → matches to blue-500 -->
<div class="bg-blue-500 text-white">
```

**Disabled** (`false`):
```html
<!-- Uses exact hex values -->
<div class="bg-[#3B82F6] text-[#FFFFFF]">
```

**Color Matching**:
- Finds closest Tailwind color using color distance
- Only matches within threshold percentage
- Falls back to arbitrary value if no close match

**Benefits**:
- Consistent with Tailwind palette
- Easier to customize theme
- Shorter class names

---

### Custom Tailwind Prefix

**Property**: `customTailwindPrefix`
**Type**: `string`
**Default**: `""` (empty string)

**Description**: Add a custom prefix to all Tailwind classes.

**Example**:

**Value**: `"tw-"`

**Output**:
```html
<div class="tw-flex tw-flex-col tw-gap-4 tw-bg-blue-500">
```

**Use Cases**:
- Avoid conflicts with existing CSS
- Namespace Tailwind classes
- Match project configuration

**Configuration**: Must match `tailwind.config.js`:
```javascript
module.exports = {
  prefix: 'tw-',
  // ... other config
};
```

---

### Base Font Size

**Property**: `baseFontSize`
**Type**: `number`
**Default**: `16`

**Description**: Base font size in pixels for rem conversion.

**Example**:

**Value**: `16`

```css
font-size: 24px; /* → 1.5rem (24 / 16) */
font-size: 16px; /* → 1rem (16 / 16) */
font-size: 12px; /* → 0.75rem (12 / 16) */
```

**Value**: `10`

```css
font-size: 24px; /* → 2.4rem (24 / 10) */
font-size: 16px; /* → 1.6rem (16 / 10) */
font-size: 12px; /* → 1.2rem (12 / 10) */
```

**Use Cases**:
- Match project's base font size
- Customize rem calculations
- Accessibility considerations

---

### Use Tailwind 4

**Property**: `useTailwind4`
**Type**: `boolean`
**Default**: `false`

**Description**: Use Tailwind v4 syntax instead of legacy syntax.

**Example**:

**Enabled** (`true`):
```html
<div class="bg-blue-500/50">
  <!-- Opacity with slash notation -->
</div>
```

**Disabled** (`false`):
```html
<div class="bg-blue-500 bg-opacity-50">
  <!-- Separate opacity class -->
</div>
```

**Differences**:
- **v4**: `bg-blue-500/50` (inline opacity)
- **Legacy**: `bg-blue-500 bg-opacity-50` (separate class)

---

### Threshold Percent

**Property**: `thresholdPercent`
**Type**: `number`
**Default**: `15`

**Description**: Maximum percentage difference for rounding values to Tailwind scale.

**Range**: `0` to `100`

**Example**:

**Value**: `15` (default)

```
18px → 16px (11% difference) ✅ Rounded
25px → 24px (4% difference) ✅ Rounded
50px → 48px (4% difference) ✅ Rounded
75px → 64px (15% difference) ✅ Rounded (at threshold)
100px → 96px (4% difference) ✅ Rounded
200px → [200px] (>15% to nearest) ❌ Use arbitrary value
```

**Value**: `10` (strict)

```
18px → [18px] (11% > 10%) ❌ Use arbitrary value
16px → 16px (0% difference) ✅ Exact match
```

**Value**: `25` (relaxed)

```
150px → 128px (15% difference) ✅ Rounded
200px → 256px (28% difference) ❌ Still too far
```

**Recommendation**:
- `0-10%`: Very strict, many arbitrary values
- `10-20%`: Balanced (recommended)
- `20-30%`: Aggressive rounding
- `30%+`: May deviate significantly from design

---

### Base Font Family

**Property**: `baseFontFamily`
**Type**: `string`
**Default**: `""`

**Description**: Override default font family in generated code.

**Example**:

**Value**: `"Inter"`

```css
font-family: Inter, sans-serif;
```

**Value**: `"Roboto"`

```css
font-family: Roboto, sans-serif;
```

**Use Cases**:
- Match project's default font
- Override Figma font with web font
- Standardize font across all generated code

---

## Framework Generation Modes

### HTML Generation Mode

**Property**: `htmlGenerationMode`
**Type**: `"html" | "jsx" | "svelte" | "styled-components"`
**Default**: `"html"`

**Options**:

| Mode | Output | Framework |
|------|--------|-----------|
| `html` | Pure HTML with inline styles | Vanilla HTML/CSS |
| `jsx` | React JSX with inline styles | React |
| `svelte` | Svelte component with scoped styles | Svelte |
| `styled-components` | Styled-components CSS-in-JS | React + styled-components |

**Example Outputs**:

**`html`**:
```html
<div style="display: flex;">
  <div>Content</div>
</div>
```

**`jsx`**:
```jsx
<div style={{display: 'flex'}}>
  <div>Content</div>
</div>
```

**`svelte`**:
```svelte
<div class="container">
  <div>Content</div>
</div>

<style>
.container {
  display: flex;
}
</style>
```

**`styled-components`**:
```jsx
const Container = styled.div`
  display: flex;
`;

<Container>
  <div>Content</div>
</Container>
```

---

### Flutter Generation Mode

**Property**: `flutterGenerationMode`
**Type**: `"snippet" | "stateless" | "fullApp"`
**Default**: `"snippet"`

**Options**:

| Mode | Output | Use Case |
|------|--------|----------|
| `snippet` | Just the widget code | Embed in existing app |
| `stateless` | StatelessWidget class | Reusable component |
| `fullApp` | Complete MaterialApp | Standalone app |

**Example Outputs**:

**`snippet`**:
```dart
Container(
  padding: EdgeInsets.all(16),
  child: Text("Hello")
)
```

**`stateless`**:
```dart
class MyWidget extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Container(
      padding: EdgeInsets.all(16),
      child: Text("Hello")
    );
  }
}
```

**`fullApp`**:
```dart
import 'package:flutter/material.dart';

void main() => runApp(MyApp());

class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      home: Scaffold(
        body: MyWidget(),
      ),
    );
  }
}

class MyWidget extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Container(
      padding: EdgeInsets.all(16),
      child: Text("Hello")
    );
  }
}
```

---

### SwiftUI Generation Mode

**Property**: `swiftUIGenerationMode`
**Type**: `"snippet" | "struct" | "preview"`
**Default**: `"snippet"`

**Options**:

| Mode | Output | Use Case |
|------|--------|----------|
| `snippet` | Just the view code | Embed in existing view |
| `struct` | View struct | Reusable component |
| `preview` | Struct + PreviewProvider | Development with preview |

**Example Outputs**:

**`snippet`**:
```swift
VStack {
  Text("Hello")
}
.padding(16)
```

**`struct`**:
```swift
struct MyView: View {
  var body: some View {
    VStack {
      Text("Hello")
    }
    .padding(16)
  }
}
```

**`preview`**:
```swift
struct MyView: View {
  var body: some View {
    VStack {
      Text("Hello")
    }
    .padding(16)
  }
}

struct MyView_Previews: PreviewProvider {
  static var previews: some View {
    MyView()
  }
}
```

---

### Compose Generation Mode

**Property**: `composeGenerationMode`
**Type**: `"snippet" | "composable" | "screen"`
**Default**: `"snippet"`

**Options**:

| Mode | Output | Use Case |
|------|--------|----------|
| `snippet` | Just the composable code | Embed in existing composable |
| `composable` | @Composable function | Reusable component |
| `screen` | Screen with Scaffold | Full screen |

**Example Outputs**:

**`snippet`**:
```kotlin
Column(modifier = Modifier.padding(16.dp)) {
  Text("Hello")
}
```

**`composable`**:
```kotlin
@Composable
fun MyScreen() {
  Column(modifier = Modifier.padding(16.dp)) {
    Text("Hello")
  }
}
```

**`screen`**:
```kotlin
@Composable
fun MyScreen() {
  Scaffold(
    topBar = {
      TopAppBar(title = { Text("My Screen") })
    }
  ) { paddingValues ->
    Column(modifier = Modifier.padding(paddingValues)) {
      Text("Hello")
    }
  }
}
```

---

## Settings API

### TypeScript Interface

```typescript
interface PluginSettings {
  // Framework selection
  framework: "HTML" | "Tailwind" | "Flutter" | "SwiftUI" | "Compose";

  // General settings
  showLayerNames: boolean;
  useColorVariables: boolean;
  embedImages: boolean;
  embedVectors: boolean;

  // Tailwind settings
  roundTailwindValues: boolean;
  roundTailwindColors: boolean;
  customTailwindPrefix: string;
  baseFontSize: number;
  useTailwind4: boolean;
  thresholdPercent: number;
  baseFontFamily: string;

  // Framework generation modes
  htmlGenerationMode: "html" | "jsx" | "svelte" | "styled-components";
  flutterGenerationMode: "snippet" | "stateless" | "fullApp";
  swiftUIGenerationMode: "snippet" | "struct" | "preview";
  composeGenerationMode: "snippet" | "composable" | "screen";
}
```

### Accessing Settings (Backend)

```typescript
// In plugin backend code
const settings = await figma.clientStorage.getAsync("settings");

// Use settings in conversion
const code = convertToCode(nodes, settings);
```

### Updating Settings (UI)

```typescript
// In plugin UI code
const updateSetting = (key: string, value: any) => {
  const newSettings = { ...settings, [key]: value };
  parent.postMessage({
    pluginMessage: {
      type: "updateSettings",
      settings: newSettings
    }
  }, "*");
};

// Usage
<Checkbox
  checked={settings.embedVectors}
  onChange={(e) => updateSetting("embedVectors", e.target.checked)}
/>
```

## Default Values

```typescript
const defaultPluginSettings: PluginSettings = {
  framework: "HTML",

  showLayerNames: true,
  useColorVariables: true,
  embedImages: false,
  embedVectors: true,

  roundTailwindValues: true,
  roundTailwindColors: true,
  customTailwindPrefix: "",
  baseFontSize: 16,
  useTailwind4: false,
  thresholdPercent: 15,
  baseFontFamily: "",

  htmlGenerationMode: "html",
  flutterGenerationMode: "snippet",
  swiftUIGenerationMode: "snippet",
  composeGenerationMode: "snippet",
};
```

## Settings Recommendations

### For Production Code

```typescript
{
  framework: "Tailwind", // or your framework
  showLayerNames: false,  // Cleaner code
  useColorVariables: true, // Design tokens
  embedImages: false,     // Use image assets
  embedVectors: true,     // Good for icons
  roundTailwindValues: true, // Consistent spacing
  roundTailwindColors: true, // Consistent colors
  thresholdPercent: 15    // Balanced rounding
}
```

### For Quick Prototypes

```typescript
{
  framework: "HTML",
  showLayerNames: true,   // Better readability
  useColorVariables: false, // Exact colors
  embedImages: true,      // Self-contained
  embedVectors: true,     // Self-contained
  roundTailwindValues: false, // Exact sizes
  roundTailwindColors: false, // Exact colors
  thresholdPercent: 0     // No rounding
}
```

### For Design Systems

```typescript
{
  framework: "Tailwind",
  showLayerNames: false,
  useColorVariables: true,  // Important!
  embedImages: false,
  embedVectors: true,
  roundTailwindValues: true,
  roundTailwindColors: true,
  customTailwindPrefix: "ds-", // Design system prefix
  thresholdPercent: 10    // Strict rounding
}
```

---

These settings provide fine-grained control over code generation, allowing you to customize output for your specific needs and workflow.
