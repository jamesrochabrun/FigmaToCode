# Framework Support

This document details how each framework is implemented in FigmaToCode and framework-specific features.

## Table of Contents
- [Overview](#overview)
- [HTML & CSS](#html--css)
- [Tailwind CSS](#tailwind-css)
- [Flutter](#flutter)
- [SwiftUI](#swiftui)
- [Jetpack Compose](#jetpack-compose)
- [Framework Comparison](#framework-comparison)

## Overview

FigmaToCode supports **5 main frameworks** with **8 output modes** total:

| Framework | Modes | Files |
|-----------|-------|-------|
| HTML | html, jsx, svelte, styled-components | `packages/backend/src/html/` |
| Tailwind | tailwind, tailwind_jsx | `packages/backend/src/tailwind/` |
| Flutter | snippet, stateless, fullApp | `packages/backend/src/flutter/` |
| SwiftUI | snippet, struct, preview | `packages/backend/src/swiftui/` |
| Compose | snippet, composable, screen | `packages/backend/src/compose/` |

## HTML & CSS

**Location**: `packages/backend/src/html/`

### Modes

#### 1. Pure HTML (`html`)

**Output**: HTML with inline styles or CSS classes

```html
<div style="display: flex; flex-direction: column; gap: 16px; background-color: #3B82F6; border-radius: 8px; padding: 16px;">
  <div style="font-size: 24px; font-weight: 600; color: #FFFFFF;">Title</div>
  <div style="font-size: 16px; color: #E5E7EB;">Description</div>
</div>
```

**Features**:
- Inline styles by default
- Can generate external CSS classes (configurable)
- Semantic HTML when possible
- CSS custom properties for color variables

#### 2. React JSX (`html_jsx`)

**Output**: JSX with inline styles

```jsx
<div style={{display: 'flex', flexDirection: 'column', gap: '16px', backgroundColor: '#3B82F6', borderRadius: '8px', padding: '16px'}}>
  <div style={{fontSize: '24px', fontWeight: 600, color: '#FFFFFF'}}>Title</div>
  <div style={{fontSize: '16px', color: '#E5E7EB'}}>Description</div>
</div>
```

**Features**:
- CamelCase CSS properties
- Object literal styles
- JSX-compatible attribute names (`className`, etc.)
- Quote handling for JSX

#### 3. Svelte (`html_svelte`)

**Output**: Svelte component with scoped styles

```svelte
<div class="container">
  <div class="title">Title</div>
  <div class="description">Description</div>
</div>

<style>
.container {
  display: flex;
  flex-direction: column;
  gap: 16px;
  background-color: #3B82F6;
  border-radius: 8px;
  padding: 16px;
}

.title {
  font-size: 24px;
  font-weight: 600;
  color: #FFFFFF;
}

.description {
  font-size: 16px;
  color: #E5E7EB;
}
</style>
```

**Features**:
- Scoped styles (default)
- Class-based styling
- Component structure
- Svelte syntax

#### 4. styled-components (`html_styled_components`)

**Output**: React with styled-components

```jsx
import styled from 'styled-components';

const Container = styled.div`
  display: flex;
  flex-direction: column;
  gap: 16px;
  background-color: #3B82F6;
  border-radius: 8px;
  padding: 16px;
`;

const Title = styled.div`
  font-size: 24px;
  font-weight: 600;
  color: #FFFFFF;
`;

const Description = styled.div`
  font-size: 16px;
  color: #E5E7EB;
`;

export const MyComponent = () => (
  <Container>
    <Title>Title</Title>
    <Description>Description</Description>
  </Container>
);
```

**Features**:
- CSS-in-JS
- Tagged template literals
- Component-based styling
- TypeScript compatible

### Implementation Details

**Builder**: `HtmlDefaultBuilder` and `HtmlTextBuilder`

**Key Functions**:

**Layout Conversion** (`htmlAutoLayout.ts`):
```typescript
const htmlAutoLayoutProps = (node: AltSceneNode) => {
  const styles: string[] = [];

  if (node.layoutMode === "HORIZONTAL") {
    styles.push("display: flex");
    styles.push("flex-direction: row");
  } else if (node.layoutMode === "VERTICAL") {
    styles.push("display: flex");
    styles.push("flex-direction: column");
  }

  // Alignment
  if (node.primaryAxisAlignItems === "CENTER") {
    styles.push("justify-content: center");
  }

  // Gap
  if (node.itemSpacing > 0) {
    styles.push(`gap: ${node.itemSpacing}px`);
  }

  return styles.join("; ");
};
```

**Color Conversion** (`htmlColor.ts`):
```typescript
const htmlColor = (paint: Paint): string => {
  if (paint.type === "SOLID") {
    const rgb = paint.color;
    const hex = rgbToHex(rgb);

    // Use CSS variable if available
    if (paint.variableColorName) {
      return `var(--${paint.variableColorName}, ${hex})`;
    }

    return hex;
  }

  if (paint.type === "GRADIENT_LINEAR") {
    return `linear-gradient(${angle}deg, ${stops.join(", ")})`;
  }

  // ... other gradient types
};
```

**Border Radius** (`htmlBorder.ts`):
```typescript
const htmlBorderRadius = (node: AltRectangleNode): string => {
  if (typeof node.cornerRadius === "number") {
    return `border-radius: ${node.cornerRadius}px`;
  }

  // Individual corners
  const {
    topLeftRadius = 0,
    topRightRadius = 0,
    bottomRightRadius = 0,
    bottomLeftRadius = 0
  } = node;

  return `border-radius: ${topLeftRadius}px ${topRightRadius}px ${bottomRightRadius}px ${bottomLeftRadius}px`;
};
```

## Tailwind CSS

**Location**: `packages/backend/src/tailwind/`

### Modes

#### 1. Tailwind HTML (`tailwind`)

```html
<div class="flex flex-col gap-4 bg-blue-500 rounded-lg p-4">
  <div class="text-2xl font-semibold text-white">Title</div>
  <div class="text-base text-gray-200">Description</div>
</div>
```

#### 2. Tailwind JSX (`tailwind_jsx`)

```jsx
<div className="flex flex-col gap-4 bg-blue-500 rounded-lg p-4">
  <div className="text-2xl font-semibold text-white">Title</div>
  <div className="text-base text-gray-200">Description</div>
</div>
```

### Implementation Details

**Builder**: `TailwindDefaultBuilder` and `TailwindTextBuilder`

**Key Features**:

**Color Matching** (`tailwindColor.ts`):
```typescript
const tailwindColor = (rgb: RGB, settings: PluginSettings): string => {
  // Convert to hex
  const hex = rgbToHex(rgb);

  if (settings.roundTailwindColors) {
    // Find nearest Tailwind color
    const nearest = nearestColorFromRgb(rgb);

    // Check if within threshold
    const distance = colorDistance(rgb, nearest.rgb);
    const threshold = settings.thresholdPercent / 100;

    if (distance < threshold) {
      return nearest.name; // e.g., "blue-500"
    }
  }

  // Use arbitrary value
  return `[${hex}]`;
};
```

**Size Conversion** (`tailwindSize.ts`):
```typescript
const tailwindWidth = (width: number, settings: PluginSettings): string => {
  if (settings.roundTailwindValues) {
    // Tailwind uses 0.25rem increments (4px if 1rem = 16px)
    const unit = 4;
    const nearestValue = nearestValueWithThreshold(
      width,
      tailwindSizeScale, // [0, 1, 2, 3, 4, 6, 8, 12, 16, ...]
      settings.thresholdPercent
    );

    if (nearestValue !== null) {
      return `w-${nearestValue / unit}`; // w-4, w-8, w-12, etc.
    }
  }

  // Use arbitrary value
  return `w-[${width}px]`;
};
```

**Tailwind v4 Support**:
```typescript
if (settings.useTailwind4) {
  // Use modern syntax
  return `bg-${color}/${opacity}`;
} else {
  // Use legacy syntax
  return `bg-${color} bg-opacity-${opacityClass}`;
}
```

**Custom Prefix**:
```typescript
const prefix = settings.customTailwindPrefix || "";
return `${prefix}flex ${prefix}flex-col`;
// With prefix "tw-" → "tw-flex tw-flex-col"
```

## Flutter

**Location**: `packages/backend/src/flutter/`

### Modes

#### 1. Snippet
Just the widget code:

```dart
Container(
  padding: EdgeInsets.all(16),
  decoration: BoxDecoration(
    color: Color(0xFF3B82F6),
    borderRadius: BorderRadius.circular(8),
  ),
  child: Column(
    crossAxisAlignment: CrossAxisAlignment.start,
    children: [
      Text(
        "Title",
        style: TextStyle(fontSize: 24, fontWeight: FontWeight.w600, color: Color(0xFFFFFFFF)),
      ),
      SizedBox(height: 16),
      Text(
        "Description",
        style: TextStyle(fontSize: 16, color: Color(0xFFE5E7EB)),
      ),
    ],
  ),
)
```

#### 2. Stateless Widget

```dart
class MyWidget extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Container(
      // ... same as snippet
    );
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
      // ... widget code
    );
  }
}
```

### Implementation Details

**Builder**: `FlutterDefaultBuilder` and `FlutterTextBuilder`

**Layout Conversion** (`flutterAutoLayout.ts`):
```typescript
const flutterAutoLayout = (node: AltSceneNode): string => {
  if (node.layoutMode === "HORIZONTAL") {
    return "Row";
  } else if (node.layoutMode === "VERTICAL") {
    return "Column";
  } else if (hasAbsoluteChildren(node)) {
    return "Stack";
  } else {
    return "Container";
  }
};
```

**Negative Spacing Handling**:
```typescript
if (node.itemSpacing < 0) {
  // Use Transform.translate for negative spacing
  return `Transform.translate(
    offset: Offset(0, ${node.itemSpacing}),
    child: ${child}
  )`;
}
```

**Color Format**:
```typescript
const flutterColor = (rgb: RGB, opacity: number = 1): string => {
  const r = Math.round(rgb.r * 255);
  const g = Math.round(rgb.g * 255);
  const b = Math.round(rgb.b * 255);
  const a = Math.round(opacity * 255);

  // ARGB format
  return `Color(0x${a.toString(16).padStart(2, '0')}${r.toString(16).padStart(2, '0')}${g.toString(16).padStart(2, '0')}${b.toString(16).padStart(2, '0')})`;
};
```

**Border Radius**:
```typescript
if (typeof node.cornerRadius === "number") {
  return `BorderRadius.circular(${node.cornerRadius})`;
} else {
  return `BorderRadius.only(
    topLeft: Radius.circular(${node.topLeftRadius}),
    topRight: Radius.circular(${node.topRightRadius}),
    bottomRight: Radius.circular(${node.bottomRightRadius}),
    bottomLeft: Radius.circular(${node.bottomLeftRadius}),
  )`;
}
```

## SwiftUI

**Location**: `packages/backend/src/swiftui/`

### Modes

#### 1. Snippet
Just the view code:

```swift
VStack(alignment: .leading, spacing: 16) {
  Text("Title")
    .font(.system(size: 24, weight: .semibold))
    .foregroundColor(.white)
  Text("Description")
    .font(.system(size: 16))
    .foregroundColor(Color(red: 0.898, green: 0.906, blue: 0.922))
}
.padding(16)
.background(Color(red: 0.231, green: 0.510, blue: 0.965))
.cornerRadius(8)
```

#### 2. Struct

```swift
struct MyView: View {
  var body: some View {
    VStack(alignment: .leading, spacing: 16) {
      // ... same as snippet
    }
  }
}
```

#### 3. Preview

```swift
struct MyView: View {
  var body: some View {
    // ... view code
  }
}

struct MyView_Previews: PreviewProvider {
  static var previews: some View {
    MyView()
  }
}
```

### Implementation Details

**Builder**: `SwiftUIDefaultBuilder` and `SwiftUITextBuilder`

**Stack Selection**:
```typescript
const swiftUIStack = (node: AltSceneNode): string => {
  if (node.layoutMode === "HORIZONTAL") {
    return "HStack";
  } else if (node.layoutMode === "VERTICAL") {
    return "VStack";
  } else {
    return "ZStack"; // for absolute positioning
  }
};
```

**10-Item Limit Workaround**:
```typescript
const swiftUIChildren = (children: AltSceneNode[]): string => {
  if (children.length <= 10) {
    return children.map(child => generateCode(child)).join("\n");
  }

  // Split into groups of 10
  const groups: AltSceneNode[][] = [];
  for (let i = 0; i < children.length; i += 10) {
    groups.push(children.slice(i, i + 10));
  }

  return groups.map(group => {
    const groupCode = group.map(child => generateCode(child)).join("\n");
    return `Group {\n${groupCode}\n}`;
  }).join("\n");
};
```

**Color Format**:
```typescript
const swiftUIColor = (rgb: RGB, opacity: number = 1): string => {
  return `Color(red: ${rgb.r.toFixed(3)}, green: ${rgb.g.toFixed(3)}, blue: ${rgb.b.toFixed(3)}, opacity: ${opacity.toFixed(3)})`;
};
```

**Modifier Chaining**:
```typescript
class SwiftUIDefaultBuilder {
  private modifiers: string[] = [];

  addModifier(modifier: string): this {
    this.modifiers.push(modifier);
    return this;
  }

  build(): string {
    let code = this.baseView;
    for (const modifier of this.modifiers) {
      code += `\n  .${modifier}`;
    }
    return code;
  }
}

// Usage
builder
  .addModifier("padding(16)")
  .addModifier("background(Color.blue)")
  .addModifier("cornerRadius(8)");
```

## Jetpack Compose

**Location**: `packages/backend/src/compose/`

### Modes

#### 1. Snippet
Just the composable code:

```kotlin
Column(
  modifier = Modifier
    .padding(16.dp)
    .background(Color(0xFF3B82F6), shape = RoundedCornerShape(8.dp)),
  verticalArrangement = Arrangement.spacedBy(16.dp)
) {
  Text(
    text = "Title",
    fontSize = 24.sp,
    fontWeight = FontWeight.SemiBold,
    color = Color.White
  )
  Text(
    text = "Description",
    fontSize = 16.sp,
    color = Color(0xFFE5E7EB)
  )
}
```

#### 2. Composable

```kotlin
@Composable
fun MyScreen() {
  Column(
    // ... same as snippet
  ) {
    // ... children
  }
}
```

#### 3. Screen

```kotlin
@Composable
fun MyScreen() {
  Scaffold(
    topBar = {
      TopAppBar(title = { Text("My Screen") })
    }
  ) { paddingValues ->
    Column(
      modifier = Modifier.padding(paddingValues)
    ) {
      // ... content
    }
  }
}
```

### Implementation Details

**Builder**: `ComposeDefaultBuilder` and `ComposeTextBuilder`

**Layout Selection**:
```typescript
const composeLayout = (node: AltSceneNode): string => {
  if (node.layoutMode === "HORIZONTAL") {
    return "Row";
  } else if (node.layoutMode === "VERTICAL") {
    return "Column";
  } else if (hasAbsoluteChildren(node)) {
    return "Box";
  } else {
    return "Box"; // default container
  }
};
```

**Modifier Chain**:
```typescript
const composeModifier = (node: AltSceneNode): string => {
  const modifiers: string[] = ["Modifier"];

  // Size
  if (node.width) {
    modifiers.push(`width(${node.width}.dp)`);
  }

  // Padding
  if (node.padding) {
    modifiers.push(`padding(${node.padding}.dp)`);
  }

  // Background
  if (node.fills) {
    modifiers.push(`background(${color}, shape = ${shape})`);
  }

  return modifiers.join("\n  .");
};
```

**Negative Spacing**:
```typescript
if (node.itemSpacing < 0) {
  // Use offset modifier
  return `Modifier.offset(y = ${node.itemSpacing}.dp)`;
}
```

**Color Format**:
```typescript
const composeColor = (rgb: RGB, opacity: number = 1): string => {
  const r = Math.round(rgb.r * 255);
  const g = Math.round(rgb.g * 255);
  const b = Math.round(rgb.b * 255);
  const a = Math.round(opacity * 255);

  return `Color(0x${a.toString(16).padStart(2, '0')}${r.toString(16).padStart(2, '0')}${g.toString(16).padStart(2, '0')}${b.toString(16).padStart(2, '0')})`;
};
```

## Framework Comparison

### Layout Systems

| Framework | Horizontal | Vertical | Absolute | Notes |
|-----------|-----------|----------|----------|-------|
| **HTML** | `flex-direction: row` | `flex-direction: column` | `position: absolute` | Flexbox/Grid |
| **Tailwind** | `flex-row` | `flex-col` | `absolute` | Utility classes |
| **Flutter** | `Row` | `Column` | `Stack` + `Positioned` | Widgets |
| **SwiftUI** | `HStack` | `VStack` | `ZStack` + `position` | View containers |
| **Compose** | `Row` | `Column` | `Box` + `offset` | Composables |

### Color Formats

| Framework | Format | Example |
|-----------|--------|---------|
| **HTML** | Hex | `#3B82F6` |
| **Tailwind** | Name/Arbitrary | `blue-500` or `[#3B82F6]` |
| **Flutter** | ARGB Hex | `Color(0xFF3B82F6)` |
| **SwiftUI** | RGB Float | `Color(red: 0.231, green: 0.510, blue: 0.965)` |
| **Compose** | ARGB Hex | `Color(0xFF3B82F6)` |

### Size Units

| Framework | Unit | Example |
|-----------|------|---------|
| **HTML** | px | `width: 200px` |
| **Tailwind** | rem-based | `w-50` (200px if 1rem=16px) |
| **Flutter** | Logical pixels | `width: 200` |
| **SwiftUI** | Points | `width: 200` |
| **Compose** | dp | `200.dp` |

### Spacing

| Framework | Syntax | Example |
|-----------|--------|---------|
| **HTML** | `gap` | `gap: 16px` |
| **Tailwind** | `gap-*` | `gap-4` (16px) |
| **Flutter** | `mainAxisSpacing` | `mainAxisSpacing: 16` |
| **SwiftUI** | `spacing:` | `spacing: 16` |
| **Compose** | `Arrangement.spacedBy` | `Arrangement.spacedBy(16.dp)` |

### Text Styling

| Framework | Font Size | Font Weight | Color |
|-----------|-----------|-------------|-------|
| **HTML** | `font-size: 16px` | `font-weight: 600` | `color: #000` |
| **Tailwind** | `text-base` | `font-semibold` | `text-black` |
| **Flutter** | `fontSize: 16` | `fontWeight: FontWeight.w600` | `color: Color(0xFF000000)` |
| **SwiftUI** | `.font(.system(size: 16))` | `weight: .semibold` | `.foregroundColor(.black)` |
| **Compose** | `fontSize = 16.sp` | `fontWeight = FontWeight.SemiBold` | `color = Color.Black` |

### Gradients

| Framework | Linear Gradient Example |
|-----------|-------------------------|
| **HTML** | `background: linear-gradient(90deg, #FF0000 0%, #0000FF 100%)` |
| **Tailwind** | `bg-gradient-to-r from-red-500 to-blue-500` |
| **Flutter** | `gradient: LinearGradient(colors: [Color(0xFFFF0000), Color(0xFF0000FF)])` |
| **SwiftUI** | `LinearGradient(colors: [.red, .blue], startPoint: .leading, endPoint: .trailing)` |
| **Compose** | `Brush.linearGradient(colors = listOf(Color.Red, Color.Blue))` |

### Shadows

| Framework | Shadow Example |
|-----------|----------------|
| **HTML** | `box-shadow: 0px 4px 8px rgba(0, 0, 0, 0.25)` |
| **Tailwind** | `shadow-md` or `shadow-[0px_4px_8px_rgba(0,0,0,0.25)]` |
| **Flutter** | `boxShadow: [BoxShadow(offset: Offset(0, 4), blurRadius: 8, color: Color(0x40000000))]` |
| **SwiftUI** | `.shadow(color: .black.opacity(0.25), radius: 4, x: 0, y: 4)` |
| **Compose** | `.shadow(elevation = 4.dp)` |

---

Each framework implementation follows consistent patterns while respecting framework-specific idioms and best practices. The builder pattern ensures maintainability and makes it easy to extend support to new frameworks.
