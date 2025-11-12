# Conversion Pipeline

This document provides a detailed explanation of how FigmaToCode converts Figma designs into code.

## Table of Contents
- [Pipeline Overview](#pipeline-overview)
- [Stage 1: Node Export](#stage-1-node-export)
- [Stage 2: Node Processing](#stage-2-node-processing)
- [Stage 3: AltNode Creation](#stage-3-altnode-creation)
- [Stage 4: Code Generation](#stage-4-code-generation)
- [Stage 5: Post-Processing](#stage-5-post-processing)
- [Parallel Operations](#parallel-operations)
- [Error Handling](#error-handling)

## Pipeline Overview

The conversion process follows a **five-stage pipeline** that transforms Figma nodes into framework-specific code:

```
User Selection → JSON Export → Node Processing → AltNode IR → Code Generation → Output
```

### High-Level Flow

```
┌─────────────────────┐
│  User selects       │
│  nodes in Figma     │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│  Export to          │
│  JSON REST v1       │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│  Process nodes      │
│  recursively        │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│  Create AltNode     │
│  intermediate rep   │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│  Generate code      │
│  (framework-specific)│
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│  Format and send    │
│  to UI              │
└─────────────────────┘
```

### Code Location
Main entry point: `packages/backend/src/code.ts`

## Stage 1: Node Export

**File**: `packages/backend/src/jsonNodeConversion.ts`

### Purpose
Convert Figma's native node structure into JSON format.

### Process

```typescript
export async function nodesToJSON(
  selectedNodes: readonly SceneNode[]
): Promise<Record<string, any>[]>
```

#### 1.1 Selection Validation
```typescript
// Get current selection
const selection = figma.currentPage.selection;

// Validate selection exists
if (selection.length === 0) {
  throw new Error("No nodes selected");
}
```

#### 1.2 JSON Export
```typescript
// Export each node as JSON REST v1
const jsonNodes = await Promise.all(
  selectedNodes.map(node =>
    node.exportAsync({ format: "JSON_REST_V1" })
  )
);
```

**Why JSON REST v1?**
- Provides complete node properties
- Includes layout and styling information
- Standardized format across Figma versions
- Contains all necessary data for conversion

#### 1.3 Group Conversion
```typescript
// Convert GROUP nodes to FRAME nodes
if (node.type === "GROUP") {
  jsonNode.type = "FRAME";
  // Groups behave like frames for conversion
}
```

**Why convert groups to frames?**
- Simplifies processing logic
- Groups and frames have similar layout behavior
- Easier to handle in code generation

### Output
Array of JSON objects representing selected nodes with all properties.

## Stage 2: Node Processing

**File**: `packages/backend/src/altNodes/altConversion/processNodes.ts`

### Purpose
Enhance JSON nodes with additional computed properties and relationships.

### Process

The `processNodePair` function recursively processes each node and its children:

```typescript
async function processNodePair(
  node: any,
  parent: any | null,
  parentRotation: number = 0
): Promise<AltNode>
```

#### 2.1 Parent Reference
```typescript
// Add bidirectional parent-child relationship
if (parent) {
  node.parent = parent;
}
```

**Why important?**
- Enables traversal up the tree
- Needed for constraint calculations
- Required for cumulative property calculations

#### 2.2 Rotation Calculation
```typescript
// Calculate cumulative rotation from all ancestors
const cumulativeRotation = parentRotation + (node.rotation || 0);
node.cumulativeRotation = cumulativeRotation;
```

**Algorithm**:
1. Start with parent's rotation
2. Add current node's rotation
3. Pass cumulative value to children
4. Handles nested rotations correctly

**Why cumulative?**
- CSS rotation is cumulative
- Figma shows individual rotation
- Must account for parent transformations

#### 2.3 Group Inlining
```typescript
// Inline GROUP children to parent
if (node.type === "GROUP" && parent) {
  // Move children to parent
  parent.children.push(...node.children);
  // Remove group from tree
  return null;
}
```

**Why inline groups?**
- Groups don't render in output
- Simplifies layout structure
- Reduces unnecessary nesting

#### 2.4 Position Calculation
```typescript
// Calculate absolute position with rotation
if (node.rotation && node.rotation !== 0) {
  const rect = calculateRectangleFromBoundingBox(
    {
      x: node.absoluteBoundingBox.x,
      y: node.absoluteBoundingBox.y,
      width: node.absoluteBoundingBox.width,
      height: node.absoluteBoundingBox.height
    },
    node.rotation
  );
  node.x = rect.left;
  node.y = rect.top;
  node.width = rect.width;
  node.height = rect.height;
}
```

**Rotation math** (from `packages/backend/src/common/commonPosition.ts`):
```typescript
function calculateRectangleFromBoundingBox(
  boundingBox: BoundingBox,
  figmaRotationDegrees: number
): RectangleStyle {
  // Convert Figma rotation to CSS rotation (negate)
  const cssRotationDegrees = -figmaRotationDegrees;
  const rotationRadians = (cssRotationDegrees * Math.PI) / 180;

  // Calculate original dimensions using rotation matrix
  const cos = Math.abs(Math.cos(rotationRadians));
  const sin = Math.abs(Math.sin(rotationRadians));

  const originalWidth =
    (boundingBox.width * cos - boundingBox.height * sin) /
    (cos * cos - sin * sin);
  const originalHeight =
    (boundingBox.height - originalWidth * sin) / cos;

  // Calculate offset
  const topLeftX = /* complex calculation */;
  const topLeftY = /* complex calculation */;

  return {
    width: originalWidth,
    height: originalHeight,
    left: boundingBox.x + topLeftX,
    top: boundingBox.y + topLeftY,
    rotation: cssRotationDegrees
  };
}
```

#### 2.5 Unique Naming
```typescript
// Generate unique sequential names
const nameCounter = new Map<string, number>();

function generateUniqueName(baseName: string): string {
  const count = nameCounter.get(baseName) || 0;
  nameCounter.set(baseName, count + 1);
  return count === 0 ? baseName : `${baseName}_${count.toString().padStart(2, '0')}`;
}

node.uniqueName = generateUniqueName(node.name);
// "Button" → "button", "button_01", "button_02", etc.
```

#### 2.6 Text Segment Extraction
```typescript
// Extract styled text segments for rich text
if (node.type === "TEXT") {
  node.styledTextSegments = await getStyledTextSegments(node);
}
```

**What are text segments?**
- Rich text with different styles in same text node
- Each segment has: text, fontSize, fontWeight, color, etc.
- Allows for mixed formatting within text

**Example**:
```
"Hello World" with "Hello" bold and "World" italic
→ [
  { text: "Hello", fontWeight: 700 },
  { text: " World", fontStyle: "italic" }
]
```

#### 2.7 Color Variable Processing
```typescript
// Process color variables asynchronously
if (node.fills) {
  await processColorVariables(node.fills);
}
if (node.strokes) {
  await processColorVariables(node.strokes);
}
```

**Process**:
1. Check if paint has `boundVariables.color`
2. Look up variable by ID: `figma.variables.getVariableByIdAsync(id)`
3. Resolve variable name
4. Store in `paint.variableColorName`

**Performance**: Async and can be slow for many variables.

#### 2.8 Icon Detection
```typescript
// Detect if node is likely an icon
node.canBeFlattened = isLikelyIcon(node);
```

**Detection algorithm** (from `packages/backend/src/common/iconDetection.ts`):
```typescript
function isLikelyIcon(node: SceneNode): boolean {
  // Check node type
  if (isVectorOrPrimitive(node)) {
    // Max 64x64 for primitives
    if (isPrimitive(node) && (node.width > 64 || node.height > 64)) {
      return false;
    }
    return true;
  }

  // Check SVG export settings
  if (hasAutoSVGExport(node)) {
    return true;
  }

  // Check container contents
  if (isContainer(node)) {
    return node.children.every(child => {
      // All children must be vectorizable
      if (isDisallowedType(child)) return false;
      return isLikelyIcon(child);
    });
  }

  return false;
}
```

#### 2.9 SVG Embedding
```typescript
// Export and attach SVG for vectorizable nodes
if (node.canBeFlattened && settings.embedVectors) {
  const svg = await node.exportAsync({ format: "SVG_STRING" });

  // Replace colors with CSS variables
  if (node.colorVariableMappings) {
    svg = replaceSVGColors(svg, node.colorVariableMappings);
  }

  node.svg = svg;
}
```

#### 2.10 Layout Defaults
```typescript
// Set default values for auto-layout properties
node.layoutMode = node.layoutMode || "NONE";
node.primaryAxisSizingMode = node.primaryAxisSizingMode || "FIXED";
node.counterAxisSizingMode = node.counterAxisSizingMode || "FIXED";
node.paddingLeft = node.paddingLeft || 0;
node.paddingRight = node.paddingRight || 0;
node.paddingTop = node.paddingTop || 0;
node.paddingBottom = node.paddingBottom || 0;
node.itemSpacing = node.itemSpacing || 0;
```

### Output
Enhanced nodes with:
- Parent references
- Cumulative rotation
- Unique names
- Text segments
- Color variable mappings
- Icon detection flags
- Embedded SVGs
- Default layout values

## Stage 3: AltNode Creation

**File**: `packages/backend/src/altNodes/alt_api_types.ts`

### Purpose
Create a framework-agnostic intermediate representation.

### AltNode Type

```typescript
type AltSceneNode = AltFrameNode | AltTextNode | AltRectangleNode | /* ... */;

interface AltNode extends BaseNode {
  // Original properties
  id: string;
  name: string;
  type: NodeType;

  // Enhanced properties
  parent: AltNode | null;
  children: AltSceneNode[];

  // Computed properties
  uniqueName: string;
  cumulativeRotation: number;
  styledTextSegments: StyledTextSegment[];
  canBeFlattened: boolean;
  svg?: string;

  // Layout properties
  x: number;
  y: number;
  width: number;
  height: number;
  isRelative: boolean;

  // Visual properties
  fills: Paint[];
  strokes: Paint[];
  effects: Effect[];
  opacity: number;
  blendMode: BlendMode;

  // Auto-layout properties
  layoutMode: "NONE" | "HORIZONTAL" | "VERTICAL";
  primaryAxisSizingMode: "FIXED" | "AUTO";
  counterAxisSizingMode: "FIXED" | "AUTO";
  paddingLeft: number;
  paddingRight: number;
  paddingTop: number;
  paddingBottom: number;
  itemSpacing: number;
  // ... more properties
}
```

### Why Intermediate Representation?

**Benefits**:
1. **Decoupling**: Separates Figma API from code generators
2. **Transformation**: Can modify without affecting original
3. **Optimization**: Add computed properties once
4. **Consistency**: Same interface for all frameworks
5. **Future-proofing**: Easier to adapt to Figma API changes

### Transformations

AltNodes enable transformations like:
- Flattening groups
- Merging similar properties
- Detecting patterns (auto-layout → flexbox)
- Optimizing structure
- Adding semantic information

## Stage 4: Code Generation

**Files**: `packages/backend/src/{framework}/{framework}Main.ts`

### Purpose
Convert AltNodes to framework-specific code.

### Process Flow

#### 4.1 Entry Point
```typescript
// Example: HTML conversion
export function htmlMain(
  sceneNode: AltSceneNode[],
  settings: PluginSettings
): string {
  return sceneNode
    .map(node => htmlWidgetGenerator(node, settings))
    .join("\n");
}
```

#### 4.2 Recursive Tree Traversal
```typescript
function htmlWidgetGenerator(
  node: AltSceneNode,
  settings: PluginSettings
): string {
  // Base case: text node
  if (node.type === "TEXT") {
    return new HtmlTextBuilder(node, settings).build();
  }

  // Recursive case: container with children
  if (node.children && node.children.length > 0) {
    const childrenCode = node.children
      .map(child => htmlWidgetGenerator(child, settings))
      .join("\n");

    return new HtmlDefaultBuilder(node, settings)
      .buildWithChildren(childrenCode);
  }

  // Leaf node
  return new HtmlDefaultBuilder(node, settings).build();
}
```

#### 4.3 Builder Pattern

Each framework uses builders with method chaining:

```typescript
class HtmlDefaultBuilder {
  private styles: string[] = [];
  private classes: string[] = [];

  constructor(
    private node: AltSceneNode,
    private settings: PluginSettings
  ) {}

  commonPositionStyles(): this {
    // Add width, height, position, padding
    this.addSize();
    this.addPosition();
    this.addPadding();
    return this;
  }

  commonShapeStyles(): this {
    // Add background, border, shadow, blur
    this.addFills();
    this.addBorder();
    this.addShadow();
    this.addBlur();
    return this;
  }

  build(additionalStyles?: string): string {
    const styleStr = this.styles.join("; ");
    return `<div style="${styleStr}">${additionalStyles || ""}</div>`;
  }
}
```

#### 4.4 Framework-Specific Conversion

**HTML Example**:
```typescript
// Auto-layout → Flexbox
if (node.layoutMode === "HORIZONTAL") {
  builder.addStyle("display", "flex");
  builder.addStyle("flex-direction", "row");
} else if (node.layoutMode === "VERTICAL") {
  builder.addStyle("display", "flex");
  builder.addStyle("flex-direction", "column");
}
```

**Tailwind Example**:
```typescript
// Auto-layout → Tailwind classes
if (node.layoutMode === "HORIZONTAL") {
  builder.addClass("flex");
  builder.addClass("flex-row");
} else if (node.layoutMode === "VERTICAL") {
  builder.addClass("flex");
  builder.addClass("flex-col");
}
```

**Flutter Example**:
```typescript
// Auto-layout → Column/Row
if (node.layoutMode === "HORIZONTAL") {
  return `Row(children: [${childrenCode}])`;
} else if (node.layoutMode === "VERTICAL") {
  return `Column(children: [${childrenCode}])`;
}
```

**SwiftUI Example**:
```typescript
// Auto-layout → HStack/VStack
if (node.layoutMode === "HORIZONTAL") {
  return `HStack { ${childrenCode} }`;
} else if (node.layoutMode === "VERTICAL") {
  return `VStack { ${childrenCode} }`;
}
```

#### 4.5 Property Conversion

Each builder has specialized methods for different properties:

**Color Conversion**:
```typescript
// HTML
"background-color: #3B82F6"

// Tailwind
"bg-blue-500"

// Flutter
"Color(0xFF3B82F6)"

// SwiftUI
"Color(red: 0.231, green: 0.510, blue: 0.965)"

// Compose
"Color(0xFF3B82F6)"
```

**Size Conversion**:
```typescript
// HTML
"width: 200px; height: 100px"

// Tailwind
"w-[200px] h-[100px]" or "w-50 h-25" (if rounded)

// Flutter
"width: 200, height: 100"

// SwiftUI
".frame(width: 200, height: 100)"

// Compose
"Modifier.width(200.dp).height(100.dp)"
```

**Spacing Conversion**:
```typescript
// HTML
"gap: 16px"

// Tailwind
"gap-4"

// Flutter
"mainAxisSpacing: 16"

// SwiftUI
"spacing: 16"

// Compose
"verticalArrangement = Arrangement.spacedBy(16.dp)"
```

### Output
Framework-specific code string.

## Stage 5: Post-Processing

**File**: Various builder files

### Purpose
Format and finalize the generated code.

### Process

#### 5.1 Indentation
```typescript
import { indentString } from "../common/indentString";

const indented = indentString(code, 2); // 2 spaces
```

**How it works**:
```typescript
function indentString(str: string, spaces: number): string {
  const indent = " ".repeat(spaces);
  return str
    .split("\n")
    .map(line => indent + line)
    .join("\n");
}
```

#### 5.2 Wrapper Addition

**HTML Full Page**:
```typescript
if (settings.htmlGenerationMode === "html" && needsWrapper) {
  code = `<!DOCTYPE html>
<html>
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
</head>
<body>
  ${code}
</body>
</html>`;
}
```

**Flutter Full App**:
```typescript
if (settings.flutterGenerationMode === "fullApp") {
  code = `import 'package:flutter/material.dart';

void main() => runApp(MyApp());

class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      home: Scaffold(
        body: ${code}
      ),
    );
  }
}`;
}
```

#### 5.3 Import Generation

**Tailwind**:
```typescript
// Add Tailwind imports if needed
if (usesArbitraryValues) {
  // Note: Arbitrary values require Tailwind v2.1+
}
```

**Flutter**:
```typescript
// Add necessary imports
const imports = new Set<string>();
if (usesCustomFonts) imports.add("package:google_fonts/google_fonts.dart");
if (usesIcons) imports.add("package:flutter/material.dart");

const importStr = Array.from(imports).map(i => `import '${i}';`).join("\n");
```

### Output
Formatted, complete code ready for use.

## Parallel Operations

The pipeline uses parallel processing for performance:

### Parallel Node Processing
```typescript
// Process multiple selected nodes in parallel
const altNodes = await Promise.all(
  jsonNodes.map(node => processNodePair(node, null, 0))
);
```

### Parallel Async Operations
```typescript
// Within a single node
await Promise.all([
  processColorVariables(node),
  renderAndAttachSVG(node),
  getStyledTextSegments(node)
]);
```

### Performance Counters
```typescript
let counterGetNodeByIdAsync = 0;
let counterGetStyledTextSegments = 0;
let counterProcessColorVariables = 0;

// Track expensive operations
performance.mark("start-conversion");
// ... conversion logic
performance.mark("end-conversion");
performance.measure("conversion", "start-conversion", "end-conversion");
```

## Error Handling

### Warning System
```typescript
const warnings: string[] = [];

// Collect warnings during conversion
if (node.type === "VECTOR" && !settings.embedVectors) {
  warnings.push("Vector is not supported without embedding");
}

if (node.fills?.some(f => f.type === "IMAGE") && !settings.embedImages) {
  warnings.push("Image fills replaced with placeholders");
}

// Send warnings to UI
figma.ui.postMessage({
  type: "codeUpdate",
  code,
  warnings
});
```

### Graceful Degradation
```typescript
// Unsupported features fallback
if (node.type === "LINE") {
  // Create simple div approximation
  return createLineApproximation(node);
}

// Missing properties fallback
const fontSize = node.fontSize || 16; // default
const lineHeight = node.lineHeight?.value || fontSize * 1.2; // default
```

### Try-Catch Blocks
```typescript
try {
  const code = convertToCode(nodes, settings);
  return code;
} catch (error) {
  console.error("Conversion error:", error);
  return `<!-- Conversion failed: ${error.message} -->`;
}
```

## Complete Example Flow

Let's trace a simple button through the entire pipeline:

### Input: Figma Button
```
FRAME "Button"
  └─ TEXT "Click me"
  Properties:
    - Size: 120x40
    - Background: #3B82F6
    - Border radius: 8px
    - Padding: 12px 24px
    - Auto-layout: horizontal
```

### Stage 1: JSON Export
```json
{
  "type": "FRAME",
  "name": "Button",
  "absoluteBoundingBox": { "x": 100, "y": 100, "width": 120, "height": 40 },
  "fills": [{ "type": "SOLID", "color": { "r": 0.231, "g": 0.510, "b": 0.965 } }],
  "cornerRadius": 8,
  "paddingLeft": 24,
  "paddingRight": 24,
  "paddingTop": 12,
  "paddingBottom": 12,
  "layoutMode": "HORIZONTAL",
  "children": [
    {
      "type": "TEXT",
      "name": "Label",
      "characters": "Click me",
      "fontSize": 16
    }
  ]
}
```

### Stage 2: Node Processing
```javascript
{
  // Original properties preserved
  ...jsonNode,

  // Added properties
  parent: null,
  uniqueName: "button",
  cumulativeRotation: 0,
  canBeFlattened: false,
  x: 100,
  y: 100,
  width: 120,
  height: 40,

  // Children processed
  children: [{
    type: "TEXT",
    uniqueName: "label",
    styledTextSegments: [{ text: "Click me", fontSize: 16, ... }]
  }]
}
```

### Stage 3: AltNode
```typescript
AltFrameNode {
  type: "FRAME",
  uniqueName: "button",
  layoutMode: "HORIZONTAL",
  fills: [SolidPaint],
  cornerRadius: 8,
  padding: { left: 24, right: 24, top: 12, bottom: 12 },
  children: [AltTextNode]
}
```

### Stage 4: Code Generation (HTML)
```html
<div style="display: flex; flex-direction: row; align-items: center; justify-content: center; padding: 12px 24px; background-color: #3B82F6; border-radius: 8px; width: 120px; height: 40px;">
  <div style="font-size: 16px; color: #000000;">Click me</div>
</div>
```

### Stage 4: Code Generation (Tailwind)
```html
<div class="flex flex-row items-center justify-center px-6 py-3 bg-blue-500 rounded-lg w-[120px] h-[40px]">
  <div class="text-base">Click me</div>
</div>
```

### Stage 4: Code Generation (Flutter)
```dart
Container(
  width: 120,
  height: 40,
  padding: EdgeInsets.symmetric(horizontal: 24, vertical: 12),
  decoration: BoxDecoration(
    color: Color(0xFF3B82F6),
    borderRadius: BorderRadius.circular(8),
  ),
  child: Row(
    mainAxisAlignment: MainAxisAlignment.center,
    children: [
      Text(
        "Click me",
        style: TextStyle(fontSize: 16),
      ),
    ],
  ),
)
```

### Stage 5: Post-Processing
Final output formatted with proper indentation and any necessary wrappers.

---

This conversion pipeline is sophisticated yet efficient, handling complex design structures while maintaining performance through parallel processing and smart optimizations.
