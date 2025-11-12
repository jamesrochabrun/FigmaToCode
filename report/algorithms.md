# Key Algorithms

This document details the important algorithms used in FigmaToCode for layout detection, color conversion, and optimization.

## Table of Contents
- [Icon Detection Algorithm](#icon-detection-algorithm)
- [Layout Detection](#layout-detection)
- [Position Calculation with Rotation](#position-calculation-with-rotation)
- [Color Conversion](#color-conversion)
- [Nearest Value Matching](#nearest-value-matching)
- [SVG Embedding with Variable Replacement](#svg-embedding-with-variable-replacement)
- [Text Segment Extraction](#text-segment-extraction)

## Icon Detection Algorithm

**File**: `packages/backend/src/common/iconDetection.ts`

### Purpose
Automatically detect which nodes can be converted to inline SVG (icons).

### Algorithm

```typescript
function isLikelyIcon(node: SceneNode): boolean
```

### Decision Tree

```
Is node a vector or primitive?
├─ YES → Is it small enough (<64x64 for primitives)?
│         ├─ YES → ✅ Return true
│         └─ NO → ❌ Return false
└─ NO → Does it have auto-SVG export setting?
         ├─ YES → ✅ Return true
         └─ NO → Is it a container?
                  ├─ YES → Check all children recursively
                  │        ├─ All vectorizable → ✅ Return true
                  │        └─ Any non-vectorizable → ❌ Return false
                  └─ NO → ❌ Return false
```

### Implementation

```typescript
function isLikelyIcon(node: SceneNode): boolean {
  // Step 1: Check if vector or primitive
  if (isVectorOrPrimitive(node)) {
    // Step 1a: Size constraint for primitives
    if (isPrimitive(node)) {
      const maxSize = 64;
      if (node.width > maxSize || node.height > maxSize) {
        return false; // Too large for an icon
      }
    }
    return true;
  }

  // Step 2: Check export settings
  if (hasAutoSVGExport(node)) {
    return true;
  }

  // Step 3: Check container contents
  if (isContainer(node)) {
    return checkContainerChildren(node);
  }

  return false;
}
```

### Helper Functions

**Check if vector or primitive**:
```typescript
function isVectorOrPrimitive(node: SceneNode): boolean {
  return [
    "VECTOR",
    "BOOLEAN_OPERATION",
    "POLYGON",
    "STAR",
    "ELLIPSE",
    "RECTANGLE"
  ].includes(node.type);
}
```

**Check if primitive** (stricter size limits):
```typescript
function isPrimitive(node: SceneNode): boolean {
  return ["RECTANGLE", "ELLIPSE", "POLYGON", "STAR"].includes(node.type);
}
```

**Check export settings**:
```typescript
function hasAutoSVGExport(node: SceneNode): boolean {
  return node.exportSettings?.some(
    setting => setting.format === "SVG" && setting.constraint?.type === "SCALE"
  ) || false;
}
```

**Check container children**:
```typescript
function checkContainerChildren(node: FrameNode | GroupNode | ComponentNode): boolean {
  // Must not have disallowed children
  const hasDisallowedChildren = node.children.some(child =>
    ["TEXT", "FRAME", "GROUP", "COMPONENT", "INSTANCE"].includes(child.type)
  );

  if (hasDisallowedChildren) {
    return false;
  }

  // All children must be vectorizable
  return node.children.every(child => isLikelyIcon(child));
}
```

### Examples

**Simple Icon** (✅ Detected):
```
VECTOR "arrow-right"
- Width: 24px, Height: 24px
→ isLikelyIcon() = true
```

**Large Illustration** (❌ Not detected):
```
VECTOR "hero-illustration"
- Width: 800px, Height: 600px
→ isLikelyIcon() = false
```

**Icon Container** (✅ Detected):
```
FRAME "icon-search"
├─ VECTOR "magnifying-glass"
└─ ELLIPSE "circle"
All children < 64x64 and vectorizable
→ isLikelyIcon() = true
```

**Mixed Container** (❌ Not detected):
```
FRAME "card"
├─ VECTOR "icon"
└─ TEXT "label"
Has TEXT child (disallowed)
→ isLikelyIcon() = false
```

## Layout Detection

**Files**: `packages/backend/src/tailwind/tailwindAutoLayout.ts` (example)

### Purpose
Convert Figma's auto-layout to framework-specific layout code.

### Auto-Layout to Flexbox/Grid

```typescript
function detectLayoutType(node: AltSceneNode): LayoutType {
  if (node.layoutMode === "HORIZONTAL") {
    return "flex-row";
  } else if (node.layoutMode === "VERTICAL") {
    return "flex-column";
  } else if (hasAbsoluteChildren(node)) {
    return "absolute-container";
  } else {
    return "block";
  }
}
```

### Property Mapping

**Tailwind Example**:

```typescript
const tailwindAutoLayoutProps = (
  node: AltSceneNode,
  autoLayout: AutoLayout
): string => {
  const classes: string[] = [];

  // Base display
  classes.push(getFlex(node, autoLayout));           // "flex" or "inline-flex"

  // Direction
  classes.push(getFlexDirection(autoLayout));        // "" or "flex-col"

  // Main axis alignment
  classes.push(getJustifyContent(autoLayout));       // "justify-start", "justify-center", etc.

  // Cross axis alignment
  classes.push(getAlignItems(autoLayout));           // "items-start", "items-center", etc.

  // Spacing
  classes.push(getGap(autoLayout));                  // "gap-4", "gap-8", etc.

  // Wrapping
  classes.push(getFlexWrap(autoLayout));             // "flex-wrap" or ""

  // Align content (multi-line)
  classes.push(getAlignContent(autoLayout));         // "content-start", etc.

  return classes.filter(Boolean).join(" ");
};
```

### Direction Mapping

```typescript
function getFlexDirection(autoLayout: AutoLayout): string {
  switch (autoLayout.layoutMode) {
    case "HORIZONTAL":
      return ""; // flex-row is default
    case "VERTICAL":
      return "flex-col";
    default:
      return "";
  }
}
```

### Alignment Mapping

```typescript
function getJustifyContent(autoLayout: AutoLayout): string {
  switch (autoLayout.primaryAxisAlignItems) {
    case "MIN":
      return "justify-start";
    case "CENTER":
      return "justify-center";
    case "MAX":
      return "justify-end";
    case "SPACE_BETWEEN":
      return "justify-between";
    default:
      return "";
  }
}

function getAlignItems(autoLayout: AutoLayout): string {
  switch (autoLayout.counterAxisAlignItems) {
    case "MIN":
      return "items-start";
    case "CENTER":
      return "items-center";
    case "MAX":
      return "items-end";
    case "BASELINE":
      return "items-baseline";
    default:
      return "";
  }
}
```

### Gap Calculation

```typescript
function getGap(autoLayout: AutoLayout): string {
  const spacing = autoLayout.itemSpacing;

  if (spacing === 0) return "";

  // Round to Tailwind scale (0, 1, 2, 4, 6, 8, 12, 16, 20, 24, ...)
  const tailwindGapScale = [0, 1, 2, 4, 6, 8, 12, 16, 20, 24, 32, 40, 48, 64];
  const nearest = nearestValue(spacing, tailwindGapScale);

  // Tailwind uses 0.25rem units (4px)
  const gapClass = nearest / 4;

  return `gap-${gapClass}`;
}
```

## Position Calculation with Rotation

**File**: `packages/backend/src/common/commonPosition.ts`

### Problem
Figma's bounding box includes rotation, but CSS applies rotation after positioning.

### Visual Explanation

```
Figma:
  [Rotated Box Bounding Box]
  ┌─────────────┐
  │   ╱╲        │  Bounding box includes rotation
  │  ╱  ╲       │
  │ ╱    ╲      │
  │╱      ╲     │
  └─────────────┘

CSS:
  [Original Box] → Rotate → [Visual Result]
  ┌──────┐
  │      │  → rotate(45deg) →    ╱╲
  │      │                       ╱  ╲
  └──────┘                      ╱    ╲
```

### Algorithm

```typescript
function calculateRectangleFromBoundingBox(
  boundingBox: { x: number; y: number; width: number; height: number },
  figmaRotationDegrees: number
): { width: number; height: number; left: number; top: number; rotation: number }
```

### Implementation

```typescript
function calculateRectangleFromBoundingBox(
  boundingBox: BoundingBox,
  figmaRotationDegrees: number
): RectangleStyle {
  // Step 1: Convert Figma rotation to CSS rotation (negate)
  const cssRotationDegrees = -figmaRotationDegrees;
  const rotationRadians = (cssRotationDegrees * Math.PI) / 180;

  // Step 2: Calculate trig values
  const cos = Math.abs(Math.cos(rotationRadians));
  const sin = Math.abs(Math.sin(rotationRadians));

  // Step 3: Calculate original dimensions
  // Solve system of equations:
  //   boundingBox.width = originalWidth * cos + originalHeight * sin
  //   boundingBox.height = originalWidth * sin + originalHeight * cos

  const originalWidth =
    (boundingBox.width * cos - boundingBox.height * sin) /
    (cos * cos - sin * sin);

  const originalHeight =
    (boundingBox.height - originalWidth * sin) / cos;

  // Step 4: Calculate rotation origin offset
  const centerX = originalWidth / 2;
  const centerY = originalHeight / 2;

  // Step 5: Calculate corner positions after rotation
  const corners = [
    { x: 0, y: 0 },                                    // Top-left
    { x: originalWidth, y: 0 },                        // Top-right
    { x: 0, y: originalHeight },                       // Bottom-left
    { x: originalWidth, y: originalHeight }            // Bottom-right
  ];

  const rotatedCorners = corners.map(corner => {
    const relativeX = corner.x - centerX;
    const relativeY = corner.y - centerY;

    return {
      x: centerX + relativeX * Math.cos(rotationRadians) - relativeY * Math.sin(rotationRadians),
      y: centerY + relativeX * Math.sin(rotationRadians) + relativeY * Math.cos(rotationRadians)
    };
  });

  // Step 6: Find bounding box of rotated corners
  const minX = Math.min(...rotatedCorners.map(c => c.x));
  const minY = Math.min(...rotatedCorners.map(c => c.y));

  // Step 7: Calculate offset
  const topLeftX = -minX;
  const topLeftY = -minY;

  return {
    width: originalWidth,
    height: originalHeight,
    left: boundingBox.x + topLeftX,
    top: boundingBox.y + topLeftY,
    rotation: cssRotationDegrees
  };
}
```

### Example

**Input**:
```
Figma node:
- Bounding box: x=100, y=100, width=141.42, height=141.42
- Rotation: 45°
```

**Calculation**:
```
1. CSS rotation = -45°
2. cos(45°) = 0.707, sin(45°) = 0.707
3. Original width = 100, height = 100
4. Offset calculation...
5. Result: left=100, top=100, width=100, height=100, rotation=-45deg
```

**Output**:
```css
position: absolute;
left: 100px;
top: 100px;
width: 100px;
height: 100px;
transform: rotate(-45deg);
```

## Color Conversion

**Files**: `packages/backend/src/common/conversionTables.ts`, `packages/backend/src/tailwind/tailwindColor.ts`

### RGB to Hex

```typescript
function rgbToHex(rgb: RGB): string {
  const r = Math.round(rgb.r * 255);
  const g = Math.round(rgb.g * 255);
  const b = Math.round(rgb.b * 255);

  return `#${r.toString(16).padStart(2, '0')}${g.toString(16).padStart(2, '0')}${b.toString(16).padStart(2, '0')}`.toUpperCase();
}
```

### Hex to RGB

```typescript
function hexToRgb(hex: string): RGB {
  const r = parseInt(hex.slice(1, 3), 16) / 255;
  const g = parseInt(hex.slice(3, 5), 16) / 255;
  const b = parseInt(hex.slice(5, 7), 16) / 255;

  return { r, g, b };
}
```

### Nearest Color Matching (Tailwind)

**Purpose**: Match arbitrary colors to closest Tailwind color.

```typescript
function nearestColorFromRgb(rgb: RGB): { name: string; value: string; rgb: RGB } {
  // Convert to 0-255 range
  const r = Math.round(rgb.r * 255);
  const g = Math.round(rgb.g * 255);
  const b = Math.round(rgb.b * 255);

  // Use nearest-color library
  const nearest = nearestColor.from(tailwindColors);
  const result = nearest({ r, g, b });

  return {
    name: result.name,      // e.g., "blue-500"
    value: result.value,    // e.g., "#3B82F6"
    rgb: result.rgb         // { r: 59, g: 130, b: 246 }
  };
}
```

### Color Distance Calculation

Uses **Euclidean distance** in RGB space:

```typescript
function colorDistance(rgb1: RGB, rgb2: RGB): number {
  const rDiff = rgb1.r - rgb2.r;
  const gDiff = rgb1.g - rgb2.g;
  const bDiff = rgb1.b - rgb2.b;

  return Math.sqrt(rDiff * rDiff + gDiff * gDiff + bDiff * bDiff);
}
```

**Normalization** for threshold comparison:

```typescript
function normalizedColorDistance(rgb1: RGB, rgb2: RGB): number {
  const distance = colorDistance(rgb1, rgb2);

  // Normalize to 0-1 range (max distance in RGB is √3)
  return distance / Math.sqrt(3);
}
```

## Nearest Value Matching

**File**: `packages/backend/src/common/numToAutoFixed.ts`

### Purpose
Round values to predefined scales (Tailwind spacing, font sizes, etc.) within a threshold.

### Algorithm

```typescript
function nearestValueWithThreshold(
  goal: number,
  array: number[],
  thresholdPercent: number
): number | null {
  // Find nearest value in array
  const nearest = array.reduce((prev, curr) => {
    return Math.abs(curr - goal) < Math.abs(prev - goal) ? curr : prev;
  });

  // Calculate percentage difference
  const percentDiff = Math.abs((nearest - goal) / goal) * 100;

  // Only return if within threshold
  if (percentDiff <= thresholdPercent) {
    return nearest;
  }

  return null; // Use exact value
}
```

### Example Usage

**Tailwind Width Rounding**:

```typescript
const tailwindWidthScale = [0, 4, 8, 12, 16, 20, 24, 32, 40, 48, 64, 80, 96, 128, 256];
const thresholdPercent = 15;

nearestValueWithThreshold(18, tailwindWidthScale, 15);
// 18 vs 16: (18-16)/18 = 11.1% < 15% → Returns 16

nearestValueWithThreshold(25, tailwindWidthScale, 15);
// 25 vs 24: (25-24)/25 = 4% < 15% → Returns 24

nearestValueWithThreshold(30, tailwindWidthScale, 15);
// 30 vs 32: (32-30)/30 = 6.7% < 15% → Returns 32

nearestValueWithThreshold(50, tailwindWidthScale, 15);
// 50 vs 48: (50-48)/50 = 4% < 15% → Returns 48

nearestValueWithThreshold(60, tailwindWidthScale, 15);
// 60 vs 64: (64-60)/60 = 6.7% < 15% → Returns 64

nearestValueWithThreshold(72, tailwindWidthScale, 15);
// 72 vs 64: (72-64)/72 = 11.1% < 15% → Returns 64
// 72 vs 80: (80-72)/72 = 11.1% < 15% → Returns 64 (closer)

nearestValueWithThreshold(100, tailwindWidthScale, 15);
// 100 vs 96: (100-96)/100 = 4% < 15% → Returns 96

nearestValueWithThreshold(150, tailwindWidthScale, 15);
// 150 vs 128: (150-128)/150 = 14.7% < 15% → Returns 128
// 150 vs 256: (256-150)/150 = 70.7% > 15% → Would return null if 256 weren't closer
// Actually returns 128 (closer)

nearestValueWithThreshold(200, tailwindWidthScale, 15);
// 200 vs 256: (256-200)/200 = 28% > 15% → Returns null (use exact [200px])
```

### Threshold Impact

```
thresholdPercent = 0: Only exact matches
thresholdPercent = 10: Very strict rounding
thresholdPercent = 15: Default (balanced)
thresholdPercent = 25: Aggressive rounding
thresholdPercent = 100: Always rounds to nearest
```

## SVG Embedding with Variable Replacement

**File**: `packages/backend/src/altNodes/altMixins.ts`

### Purpose
Embed SVG inline while preserving color variable references.

### Algorithm

```typescript
async function renderAndAttachSVG(node: AltSceneNode): Promise<void> {
  // Step 1: Export as SVG
  const svg = await node.exportAsync({ format: "SVG_STRING" });

  // Step 2: Process color variables
  if (node.colorVariableMappings && node.colorVariableMappings.size > 0) {
    // Step 3: Replace colors with CSS variables
    const processedSvg = svg.replace(
      /(fill|stroke)="([^"]*)"/g,
      (match, attribute, color) => {
        // Check if this color has a variable mapping
        const mapping = node.colorVariableMappings.get(color);

        if (mapping) {
          // Replace with CSS variable
          return `${attribute}="var(--${mapping.variableName}, ${color})"`;
        }

        // Keep original color
        return match;
      }
    );

    node.svg = processedSvg;
  } else {
    node.svg = svg;
  }
}
```

### Example

**Input SVG**:
```xml
<svg>
  <rect fill="#3B82F6" />
  <circle stroke="#10B981" />
</svg>
```

**Color Mappings**:
```
#3B82F6 → primary-500
#10B981 → success-500
```

**Output SVG**:
```xml
<svg>
  <rect fill="var(--primary-500, #3B82F6)" />
  <circle stroke="var(--success-500, #10B981)" />
</svg>
```

**Benefits**:
- Preserves design system color tokens
- Fallback to original color if variable not defined
- Enables theming of embedded SVGs

## Text Segment Extraction

**File**: `packages/backend/src/altNodes/altConversion/convertTextToAltNode.ts`

### Purpose
Extract rich text formatting for mixed-style text.

### Algorithm

```typescript
async function getStyledTextSegments(node: TextNode): Promise<StyledTextSegment[]> {
  const segments: StyledTextSegment[] = [];
  const text = node.characters;

  // Iterate through each character
  for (let i = 0; i < text.length; ) {
    // Get properties at this position
    const fontSize = node.getRangeFontSize(i, i + 1);
    const fontWeight = node.getRangeFontWeight(i, i + 1);
    const fontFamily = node.getRangeFontName(i, i + 1);
    const textDecoration = node.getRangeTextDecoration(i, i + 1);
    const textCase = node.getRangeTextCase(i, i + 1);
    const fillStyleId = node.getRangeFillStyleId(i, i + 1);

    // Find end of segment with same properties
    let j = i + 1;
    while (j < text.length && propertiesMatch(node, i, j)) {
      j++;
    }

    // Extract segment
    segments.push({
      text: text.substring(i, j),
      fontSize: fontSize,
      fontWeight: fontWeight,
      fontFamily: fontFamily,
      textDecoration: textDecoration,
      textCase: textCase,
      fillStyleId: fillStyleId,
      // ... other properties
    });

    i = j;
  }

  return segments;
}
```

### Example

**Input**: "Hello **World**" (World is bold)

**Output**:
```typescript
[
  {
    text: "Hello ",
    fontSize: 16,
    fontWeight: 400,
    fontFamily: { family: "Inter", style: "Regular" }
  },
  {
    text: "World",
    fontSize: 16,
    fontWeight: 700,
    fontFamily: { family: "Inter", style: "Bold" }
  }
]
```

**Code Generation** (HTML):
```html
<div>
  <span style="font-size: 16px; font-weight: 400;">Hello </span>
  <span style="font-size: 16px; font-weight: 700;">World</span>
</div>
```

---

These algorithms form the core intelligence of FigmaToCode, enabling accurate conversion from design to code while respecting framework-specific idioms and user preferences.
