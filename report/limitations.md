# Limitations

This document outlines known limitations, constraints, and unsupported features of the FigmaToCode plugin.

## Table of Contents
- [Design Limitations](#design-limitations)
- [Framework-Specific Limitations](#framework-specific-limitations)
- [Visual Property Limitations](#visual-property-limitations)
- [Performance Limitations](#performance-limitations)
- [Technical Constraints](#technical-constraints)

## Design Limitations

### Unsupported Node Types

| Node Type | Support | Notes |
|-----------|---------|-------|
| **LINE** | ⚠️ Limited | Basic support only, may not convert accurately |
| **STAR** | ⚠️ Limited | Basic support, complex shapes may fail |
| **POLYGON** | ⚠️ Limited | Basic support, complex shapes may fail |
| **SLICE** | ❌ None | Completely ignored during conversion |

### Vector Graphics

**Status**: ⚠️ **Partial Support**

**Supported**:
- ✅ SVG embedding (HTML/Tailwind only)
- ✅ Icon detection and conversion
- ✅ Simple vector shapes

**Limitations**:
- ❌ No native vector support in Flutter
- ❌ No native vector support in SwiftUI
- ❌ No native vector support in Compose
- ⚠️ Complex boolean operations may not convert accurately
- ⚠️ Vector with effects may lose fidelity
- ⚠️ Very complex paths may produce large SVG code

**Workarounds**:
- Export vectors as separate SVG files manually
- Use icon libraries instead of embedded vectors
- Simplify complex vector operations in Figma
- Enable `embedVectors: true` for HTML/Tailwind

### Image Handling

**Status**: ⚠️ **Partial Support**

**Supported**:
- ✅ Image placeholder generation
- ✅ Base64 inline embedding (HTML only)
- ✅ Aspect ratio preservation

**Limitations**:
- ❌ No automatic asset management
- ❌ No image optimization
- ❌ No responsive image support
- ⚠️ Base64 embedding increases file size significantly
- ⚠️ No support for image transformations
- ⚠️ Image fills converted to placeholders by default

**Workarounds**:
- Enable `embedImages: true` for inline base64 (HTML)
- Manually export images and reference them
- Use placeholder URLs and replace later
- Consider using Figma's export functionality separately

### Layout Constraints

**Status**: ⚠️ **Simplified**

Figma's constraint system is more sophisticated than CSS/framework equivalents.

**Limitations**:
- ❌ SCALE constraints not supported
- ❌ MIN/MAX constraints simplified
- ⚠️ Complex constraint combinations simplified to common patterns
- ⚠️ Constraints within constraints may not work as expected

**Example**:
```
Figma: "Left & Right" constraints with padding
Output: width: 100% with padding (simplified)
```

### Component Features

**Status**: ⚠️ **Limited Support**

**Supported**:
- ✅ Basic component instances
- ✅ Component nesting
- ✅ Component overrides (visual only)

**Limitations**:
- ❌ Component properties/variants not fully supported
- ❌ Interactive component states ignored
- ❌ Component instance swapping not supported
- ⚠️ Components treated as regular groups
- ⚠️ No component prop generation

## Framework-Specific Limitations

### HTML/Tailwind

**Blend Modes**:
- ⚠️ Limited browser support
- Some blend modes may not work in all browsers
- Fallback to normal blend mode in unsupported browsers

**Inner Shadows**:
- ❌ Not supported in Tailwind (no utility class)
- ✅ Supported in HTML with `box-shadow: inset`

**Gradients**:
- ⚠️ Conic gradients have limited browser support
- ⚠️ Complex gradient stops may simplify

### Flutter

**Stack Item Limit**:
- ⚠️ No enforced limit, but deeply nested stacks can be slow

**Negative Spacing**:
- ❌ Not supported
- Fallback: Uses `Transform.translate` or `SizedBox.shrink`

**Text Overflow**:
- ⚠️ Limited overflow options compared to CSS
- Only `ellipsis`, `clip`, `fade`, `visible`

**Shadows**:
- ⚠️ Only one shadow per container
- Multiple shadows combined or dropped

**Images**:
- ⚠️ Image fills converted to placeholders
- No automatic asset integration

**Example limitation**:
```dart
// Figma: Multiple shadows
// Output: Only first shadow preserved
Container(
  decoration: BoxDecoration(
    boxShadow: [
      BoxShadow(/* only first shadow */)
    ]
  )
)
```

### SwiftUI

**Stack Item Limit**:
- ⚠️ **Maximum 10 items per Stack**
- Exceeding 10 items causes compiler errors
- **Workaround**: Plugin automatically uses `Group()` for >10 items

```swift
// Automatic workaround for >10 items
VStack {
  Group {
    Item1()
    Item2()
    // ... up to 10
  }
  Group {
    Item11()
    // ... next batch
  }
}
```

**Gradients**:
- ⚠️ Gradient stops limited to SwiftUI's API
- Complex gradients may simplify

**Shadows**:
- ⚠️ Only basic shadow support
- No multiple shadows
- Limited shadow customization

**Images**:
- ⚠️ Image fills converted to placeholders
- Requires manual asset management

### Jetpack Compose

**Negative Spacing**:
- ❌ Not supported
- Uses workarounds with `offset()`

**Wrap Layout**:
- ⚠️ Limited support
- Requires FlowRow/FlowColumn (Material 3)

**Images**:
- ⚠️ Image fills not fully supported
- Requires manual drawable integration

**Shadows**:
- ⚠️ Only elevation-based shadows
- No custom shadow offsets until API 28+

## Visual Property Limitations

### Text Properties

**Unsupported**:
- ❌ OpenType features (except superscript/subscript)
- ❌ Variable fonts
- ❌ Advanced typography features
- ❌ Text paths/shapes
- ⚠️ Vertical text alignment (approximate)
- ⚠️ List styles

**Font Mapping**:
- ⚠️ Font family names may need adjustment
- ⚠️ Font weights may not match exactly across platforms
- ⚠️ Custom fonts require manual installation

**Example**:
```
Figma: "SF Pro Display" → SwiftUI: ".system"
Figma: "Inter" → May need font files on Android/iOS
```

### Effects Limitations

**Background Blur** (`backdrop-filter`):
- ⚠️ Limited browser support (not supported in Firefox)
- ✅ Supported in Chrome, Safari, Edge
- ⚠️ May have performance impact

**Layer Blur**:
- ⚠️ Figma blur ÷ 2 for CSS approximation
- Blur intensity may not match exactly

**Blend Modes**:
```
✅ Full support: multiply, screen, overlay, darken, lighten
⚠️ Partial support: color-dodge, color-burn, hard-light, soft-light
❌ Limited support: difference, exclusion, hue, saturation, color, luminosity
```

**Multiple Effects**:
- ⚠️ Effect order may matter
- ⚠️ Some effect combinations may not work as expected

### Rotation Limitations

**Supported**:
- ✅ Basic rotation with CSS transforms
- ✅ Cumulative rotation calculation

**Limitations**:
- ⚠️ Rotation + effects may have issues
- ⚠️ Rotation in Flutter uses Transform widget (may affect layout)
- ⚠️ 3D rotations not supported
- ⚠️ Rotation origin may differ from Figma

### Gradient Limitations

**Supported**:
- ✅ Linear gradients
- ✅ Radial gradients
- ✅ Angular/conic gradients

**Limitations**:
- ⚠️ Gradient angle calculation may differ slightly
- ⚠️ Gradient opacity stops may combine with color stops
- ⚠️ Very complex gradients (>10 stops) may be verbose
- ⚠️ Gradient on borders limited in some frameworks

**Framework support**:
```
HTML/Tailwind: Full gradient support
Flutter: LinearGradient, RadialGradient, SweepGradient
SwiftUI: LinearGradient, RadialGradient, AngularGradient
Compose: Brush.linearGradient, Brush.radialGradient, Brush.sweepGradient
```

## Performance Limitations

### Processing Speed

**Fast** (<1 second):
- Single node conversion
- Simple selections (1-10 nodes)
- Text-only nodes

**Medium** (1-3 seconds):
- Medium selections (10-50 nodes)
- Nodes with vectors
- Rich text elements

**Slow** (3+ seconds):
- Large selections (50+ nodes)
- Deep nesting (>10 levels)
- Many SVG exports
- Complex text with many segments

**Very Slow** (10+ seconds):
- Entire page conversions (100+ nodes)
- Many complex vectors
- Deeply nested auto-layouts

### Memory Limitations

**Potential Issues**:
- Very large selections may cause memory issues
- Many embedded SVGs increase memory usage
- Base64 image embedding significantly increases memory
- Deep recursion may hit call stack limits

**Recommendations**:
- Select smaller portions of designs
- Avoid embedding many large images
- Break up very large designs into components
- Use vector embedding sparingly

### API Call Limitations

**Async operations** that can be slow:
- `node.exportAsync()` - SVG export (200-500ms each)
- `figma.variables.getVariableByIdAsync()` - Variable lookup
- `node.exportAsync({ format: "JSON_REST_V1" })` - JSON export

**Performance counters** track these calls:
```typescript
counterGetNodeByIdAsync
counterGetStyledTextSegments
counterProcessColorVariables
```

## Technical Constraints

### Figma Plugin API Constraints

**Network Access**:
- ❌ No network access (`"allowedDomains": ["none"]`)
- ❌ Cannot fetch external resources
- ❌ Cannot upload to servers

**Storage**:
- ⚠️ Limited to `figma.clientStorage` (no size limit documented)
- Settings stored locally per user

**Permissions**:
- ✅ Read-only access to nodes
- ❌ Cannot modify the design
- ❌ Cannot access user information

### Code Generation Constraints

**Code Quality**:
- ⚠️ Generated code may not follow all best practices
- ⚠️ May not be perfectly semantic HTML
- ⚠️ May have redundant styles
- ⚠️ Class names are auto-generated (not semantic)
- ⚠️ No accessibility attributes (ARIA)

**Code Size**:
- ⚠️ Large selections generate large code files
- ⚠️ Inline styles increase HTML size
- ⚠️ Embedded SVGs can be verbose
- ⚠️ No automatic code minification

**Indentation**:
- Fixed indentation (2 spaces)
- No customizable indentation settings

### Browser/Platform Compatibility

**HTML Output**:
- Targets modern browsers (ES6+)
- May use features not supported in IE11
- Some CSS features require vendor prefixes

**Tailwind Output**:
- Requires Tailwind CSS v2.0+ or v4.0+
- Arbitrary values require Tailwind v2.1+
- Some utilities require specific Tailwind versions

**Flutter Output**:
- Targets Flutter 2.0+
- May use newer widgets/APIs
- Null safety syntax

**SwiftUI Output**:
- Targets iOS 13+ / macOS 10.15+
- Uses modern SwiftUI syntax
- May not work on older OS versions

**Compose Output**:
- Targets Compose 1.0+
- Material 3 features may require newer versions
- Requires specific Kotlin versions

## Specific Feature Limitations

### Auto-Layout Edge Cases

**Not Fully Supported**:
- ❌ Negative spacing in Flutter/Compose
- ⚠️ Absolute positioning within auto-layout (uses Stack/Box)
- ⚠️ Complex wrapping behavior
- ⚠️ Baseline alignment

**Workarounds Applied**:
```dart
// Negative spacing → Transform.translate
Transform.translate(
  offset: Offset(-8, 0),
  child: ...
)

// Absolute in auto-layout → Stack
Stack(
  children: [
    Column(...), // auto-layout children
    Positioned(...) // absolute child
  ]
)
```

### Color Variable Limitations

**Supported**:
- ✅ Solid color variables
- ✅ Variable names in code
- ✅ Fallback values

**Limitations**:
- ⚠️ Gradient variables may not preserve variable reference
- ⚠️ Variable aliases may flatten
- ⚠️ Variable scopes not reflected in code
- ⚠️ Collections not automatically grouped

### Responsive Design

**Limitations**:
- ❌ No automatic responsive breakpoints
- ❌ No media queries generated
- ❌ No responsive image srcsets
- ⚠️ Fixed pixel values (unless using rem with baseFontSize)

**What to do**:
- Manually add responsive breakpoints
- Use framework-specific responsive utilities
- Convert fixed values to relative units

### Accessibility

**Current State**:
- ❌ No ARIA attributes generated
- ❌ No semantic HTML guarantees
- ❌ No alt text for images
- ❌ No focus management
- ❌ No keyboard navigation support

**Recommendations**:
- Manually add accessibility attributes
- Review generated code for semantic structure
- Add ARIA labels where needed
- Test with screen readers

## Known Issues & Gotchas

### Text Truncation
- Setting truncation in Figma generates appropriate code
- Requires explicit setting in Figma (not automatic)

### Layer Names
- Special characters in layer names may cause issues
- Reserved keywords (class, div, etc.) not escaped
- Use `showLayerNames: false` to avoid conflicts

### Group Flattening
- Groups are automatically inlined
- Group names lost in conversion
- May affect intended structure

### Component Instances
- Treated as regular containers
- Instance-specific overrides only affect visual output
- No component prop binding

### Export Settings
- Node export settings (SVG, PNG) affect conversion
- Setting node as "Exportable as SVG" marks it as icon
- May cause unexpected SVG embedding

### Color Precision
- RGB values converted between [0-1] and [0-255]
- May have slight rounding differences
- Hex colors are rounded to nearest integer

## Workarounds & Best Practices

### For Better Results:

1. **Use Auto-Layout**: Better conversion than absolute positioning
2. **Avoid Deep Nesting**: Keep hierarchy shallow (<10 levels)
3. **Use Color Variables**: Better maintainability
4. **Name Layers Clearly**: Helps with debugging
5. **Simplify Vectors**: Complex vectors → simple shapes when possible
6. **Export Images Separately**: Better than base64 embedding
7. **Select Smaller Portions**: Better performance and quality
8. **Use Components**: Reuse designs (even if conversion is basic)
9. **Test in Target Framework**: Generated code may need adjustments
10. **Review Generated Code**: Always review and refine

### When Plugin Can't Help:

- **Interactions/Animations**: Use framework-specific animation libraries
- **Complex Logic**: Add manually after conversion
- **State Management**: Add framework state management
- **Routing**: Add navigation/routing manually
- **API Integration**: Add data fetching manually
- **Forms**: Add form validation and handling
- **Accessibility**: Add ARIA attributes manually
- **Performance**: Optimize generated code as needed

---

While FigmaToCode handles many common design-to-code scenarios, understanding these limitations helps set realistic expectations and guides better design practices in Figma for optimal code generation.
