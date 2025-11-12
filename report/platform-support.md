# Platform Support

This document details what platforms FigmaToCode supports and whether it's specific to web development.

## Table of Contents
- [Overview](#overview)
- [Web Platform Support](#web-platform-support)
- [Mobile Platform Support](#mobile-platform-support)
- [Desktop Platform Support](#desktop-platform-support)
- [Cross-Platform Summary](#cross-platform-summary)

## Overview

**FigmaToCode is NOT web-specific.** It supports both web and mobile/desktop development frameworks.

### Platform Coverage

| Platform | Frameworks | Status |
|----------|-----------|--------|
| **Web** | HTML, React, Svelte, Tailwind, styled-components | ✅ Full Support |
| **iOS** | SwiftUI | ✅ Full Support |
| **Android** | Flutter, Jetpack Compose | ✅ Full Support |
| **Cross-Platform Mobile** | Flutter | ✅ Full Support |
| **macOS** | SwiftUI | ✅ Supported (same as iOS) |
| **Linux/Windows Desktop** | Flutter | ✅ Supported (same as mobile Flutter) |

## Web Platform Support

### Supported Web Frameworks

#### 1. HTML + CSS
**Target**: Modern web browsers (Chrome, Firefox, Safari, Edge)

**Output Formats**:
- Pure HTML with inline styles
- HTML with CSS classes
- Semantic HTML structure

**Browser Compatibility**:
- ✅ Modern browsers (Chrome 90+, Firefox 88+, Safari 14+, Edge 90+)
- ⚠️ Some features require modern CSS (Grid, Flexbox, backdrop-filter)
- ❌ No IE11 support

**Example Output**:
```html
<div style="display: flex; flex-direction: column; gap: 16px; background-color: #3B82F6; border-radius: 8px; padding: 16px;">
  <div style="font-size: 24px; font-weight: 600; color: #FFFFFF;">Title</div>
  <div style="font-size: 16px; color: #E5E7EB;">Description</div>
</div>
```

#### 2. React (JSX)
**Target**: React 16.8+ (Hooks supported)

**Output Format**: JSX with inline styles

**Features**:
- CamelCase CSS properties
- JSX-compatible syntax
- React components structure
- Hooks-compatible (though not generated)

**Example Output**:
```jsx
<div style={{display: 'flex', flexDirection: 'column', gap: '16px', backgroundColor: '#3B82F6', borderRadius: '8px', padding: '16px'}}>
  <div style={{fontSize: '24px', fontWeight: 600, color: '#FFFFFF'}}>Title</div>
  <div style={{fontSize: '16px', color: '#E5E7EB'}}>Description</div>
</div>
```

#### 3. Svelte
**Target**: Svelte 3+

**Output Format**: Svelte component with scoped styles

**Features**:
- Scoped CSS
- Svelte component structure
- Class-based styling

**Example Output**:
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
</style>
```

#### 4. styled-components
**Target**: styled-components v5+

**Output Format**: CSS-in-JS with tagged template literals

**Features**:
- Tagged template literals
- TypeScript compatible
- React integration

**Example Output**:
```jsx
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
```

#### 5. Tailwind CSS
**Target**: Tailwind CSS v2.0+ or v4.0+

**Output Formats**:
- HTML + Tailwind classes
- JSX + Tailwind classes

**Features**:
- Utility-first classes
- Arbitrary values for custom properties
- Custom prefix support
- Color matching to Tailwind palette
- Tailwind v4 syntax support

**Example Output**:
```html
<div class="flex flex-col gap-4 bg-blue-500 rounded-lg p-4">
  <div class="text-2xl font-semibold text-white">Title</div>
  <div class="text-base text-gray-200">Description</div>
</div>
```

### Web-Specific Features

**CSS Features Used**:
- Flexbox (`display: flex`)
- CSS Grid (`display: grid`)
- CSS Custom Properties (`var(--color-name)`)
- backdrop-filter (background blur)
- transform (rotation, translation)
- box-shadow (shadows)
- linear-gradient, radial-gradient (gradients)

**Browser Limitations**:
- backdrop-filter: Not supported in Firefox
- Some blend modes: Limited support
- CSS Grid: Supported in modern browsers only

## Mobile Platform Support

### iOS - SwiftUI

**Target Platform**: iOS 13+, macOS 10.15+

**Output Modes**:
1. **Snippet**: Just the view code
2. **Struct**: Full SwiftUI View struct
3. **Preview**: With PreviewProvider

**Features**:
- HStack, VStack, ZStack layouts
- SwiftUI modifiers
- SF Symbols support
- System fonts
- Color and gradient support

**Platform-Specific Considerations**:
- Uses `.system` for default fonts
- Colors use SwiftUI Color API
- Spacing uses SwiftUI's spacing system
- Maximum 10 items per Stack (uses Group workaround)

**Example Output**:
```swift
struct MyView: View {
  var body: some View {
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
  }
}
```

**iOS Version Support**:
- ✅ iOS 13+: Full support
- ⚠️ Some modifiers may require iOS 14+
- ⚠️ Advanced features may require iOS 15+

### Android - Flutter

**Target Platform**: Android, iOS, Web, Desktop (Windows, macOS, Linux)

**Output Modes**:
1. **Snippet**: Just the widget code
2. **Stateless**: StatelessWidget class
3. **Full App**: Complete MaterialApp

**Features**:
- Column, Row, Stack layouts
- Material Design widgets
- Custom fonts support
- Flexible and Expanded widgets
- Padding and sizing

**Platform-Specific Considerations**:
- Uses Material Design by default
- Pixel-based sizing (dp on Android)
- Colors use Color(0xFFRRGGBB) format
- No direct vector support (requires SVG packages)

**Example Output**:
```dart
class MyWidget extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Container(
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
            style: TextStyle(
              fontSize: 24,
              fontWeight: FontWeight.w600,
              color: Color(0xFFFFFFFF),
            ),
          ),
          SizedBox(height: 16),
          Text(
            "Description",
            style: TextStyle(
              fontSize: 16,
              color: Color(0xFFE5E7EB),
            ),
          ),
        ],
      ),
    );
  }
}
```

**Platform Coverage**:
- ✅ Android: Primary target
- ✅ iOS: Full support
- ✅ Web: Supported (Flutter Web)
- ✅ Desktop: Windows, macOS, Linux

### Android - Jetpack Compose

**Target Platform**: Android 5.0+ (API 21+)

**Output Modes**:
1. **Snippet**: Just the composable code
2. **Composable**: Full @Composable function
3. **Screen**: Screen-level composable with Scaffold

**Features**:
- Column, Row, Box layouts
- Modifier system
- Material Design 3
- Compose UI components
- Kotlin syntax

**Platform-Specific Considerations**:
- Uses dp for sizing
- Colors use Color(0xFFRRGGBB)
- Material 3 theming
- Requires Kotlin

**Example Output**:
```kotlin
@Composable
fun MyScreen() {
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
}
```

**Android Version Support**:
- ✅ API 21+: Full support
- ⚠️ Some features require API 23+
- ⚠️ Advanced effects may require API 28+

## Desktop Platform Support

### macOS - SwiftUI

**Same as iOS SwiftUI**, with considerations:
- ✅ macOS 10.15+
- ✅ Full SwiftUI support
- ⚠️ Some iOS-specific modifiers may not work
- ⚠️ AppKit differences from UIKit

### Windows/Linux - Flutter

**Same as Flutter mobile**, with considerations:
- ✅ Windows 7+
- ✅ Modern Linux distributions
- ⚠️ Desktop-specific layouts may need adjustment
- ⚠️ Mouse/keyboard interactions not generated

## Cross-Platform Summary

### Framework Platform Matrix

| Framework | Web | iOS | Android | macOS | Windows | Linux |
|-----------|-----|-----|---------|-------|---------|-------|
| **HTML/CSS** | ✅ | ❌ | ❌ | ❌ | ❌ | ❌ |
| **React** | ✅ | ⚠️* | ⚠️* | ❌ | ❌ | ❌ |
| **Tailwind** | ✅ | ❌ | ❌ | ❌ | ❌ | ❌ |
| **Svelte** | ✅ | ❌ | ❌ | ❌ | ❌ | ❌ |
| **styled-components** | ✅ | ❌ | ❌ | ❌ | ❌ | ❌ |
| **Flutter** | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |
| **SwiftUI** | ❌ | ✅ | ❌ | ✅ | ❌ | ❌ |
| **Compose** | ❌ | ❌ | ✅ | ❌ | ❌ | ❌ |

*React Native possible but not the generated format

### Multi-Platform Workflows

#### Scenario 1: Web + Mobile App
**Recommended**: Use separate conversions
- Web: Tailwind or React
- iOS: SwiftUI
- Android: Compose or Flutter

#### Scenario 2: Cross-Platform App
**Recommended**: Flutter
- Single codebase for all platforms
- Consistent design across platforms
- Web support included

#### Scenario 3: Web Only
**Recommended**: Tailwind or React
- Best web development experience
- Tailwind for utility-first approach
- React for component-based approach

#### Scenario 4: iOS Only
**Recommended**: SwiftUI
- Native iOS development
- Best performance
- Platform-specific features

#### Scenario 5: Android Only
**Recommended**: Jetpack Compose
- Modern Android development
- Material Design 3
- Kotlin benefits

## Not Web-Specific Conclusion

**FigmaToCode is a multi-platform design-to-code tool**, not limited to web development.

### Platform Distribution of Support:

**Web-Only Frameworks**: 5 output modes
- HTML, React JSX, Svelte, styled-components, Tailwind

**Mobile/Native Frameworks**: 3 frameworks
- Flutter (cross-platform)
- SwiftUI (Apple platforms)
- Jetpack Compose (Android)

**Total Coverage**:
- **Web**: ✅ Excellent
- **Mobile**: ✅ Excellent (iOS + Android)
- **Desktop**: ✅ Good (via Flutter and SwiftUI)

### Use Case Distribution

Based on framework support, FigmaToCode is suitable for:
- 🌐 **Web development**: ~40% (5/8 output modes)
- 📱 **Mobile development**: ~40% (Flutter, SwiftUI, Compose)
- 🖥️ **Desktop development**: ~20% (Flutter desktop, macOS SwiftUI)

### Plugin Environment

The **plugin itself runs in Figma**, which is:
- **Web-based**: Figma runs in browser or Electron app
- **Platform-agnostic**: Generates code for any platform
- **Cloud-connected**: Figma design files in cloud

But the **generated code targets**:
- Web browsers
- iOS devices
- Android devices
- Desktop applications

---

**Conclusion**: FigmaToCode is a **universal design-to-code converter** that supports web, mobile, and desktop development equally well, with no preference or limitation to web-only platforms.
