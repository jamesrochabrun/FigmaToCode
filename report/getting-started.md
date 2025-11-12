# Getting Started

This guide helps developers who want to contribute to or understand the FigmaToCode codebase.

## Table of Contents
- [Prerequisites](#prerequisites)
- [Project Setup](#project-setup)
- [Development Workflow](#development-workflow)
- [Project Structure](#project-structure)
- [Making Changes](#making-changes)
- [Testing Your Changes](#testing-your-changes)
- [Adding a New Framework](#adding-a-new-framework)
- [Common Tasks](#common-tasks)
- [Troubleshooting](#troubleshooting)

## Prerequisites

### Required Software

1. **Node.js** (v18 or higher)
   ```bash
   node --version  # Should be 18.x or higher
   ```

2. **pnpm** (v9.14.4)
   ```bash
   npm install -g pnpm@9.14.4
   pnpm --version
   ```

3. **Figma Desktop App**
   - Download from [figma.com](https://www.figma.com/downloads/)
   - Required for plugin development

4. **Code Editor**
   - VS Code recommended
   - TypeScript support required

### Recommended VS Code Extensions

- **ESLint** - Code linting
- **Prettier** - Code formatting
- **TypeScript Vue Plugin** - Better TypeScript support
- **Tailwind CSS IntelliSense** - Tailwind class completion

## Project Setup

### 1. Clone the Repository

```bash
git clone https://github.com/bernaferrari/FigmaToCode.git
cd FigmaToCode
```

### 2. Install Dependencies

```bash
pnpm install
```

This installs all dependencies across all packages in the monorepo.

### 3. Build the Project

```bash
pnpm build
```

This builds all packages in the correct order.

### 4. Start Development Mode

```bash
pnpm dev
```

This starts:
- Plugin backend in watch mode
- Plugin UI with hot reload
- Debug UI at `http://localhost:3000`

## Development Workflow

### Running the Plugin in Figma

1. **Build or start dev mode**:
   ```bash
   pnpm dev  # For development with watch mode
   # OR
   pnpm build  # For one-time build
   ```

2. **Load plugin in Figma**:
   - Open Figma Desktop App
   - Go to Menu → Plugins → Development → Import plugin from manifest
   - Navigate to `FigmaToCode/manifest.json`
   - Click "Open"

3. **Run the plugin**:
   - Create or open a Figma file
   - Right-click → Plugins → Development → Figma to Code
   - The plugin UI will open

4. **See changes in real-time** (dev mode only):
   - Backend changes: Save file, plugin reloads automatically
   - UI changes: Changes appear immediately (HMR)

### Using the Debug UI

The debug UI provides a convenient way to develop and test without Figma:

1. **Start debug mode**:
   ```bash
   pnpm dev
   ```

2. **Open browser**:
   ```
   http://localhost:3000
   ```

3. **Features**:
   - Browse all UI components
   - Test with sample data
   - View JSON output
   - No need to reload Figma

## Project Structure

```
FigmaToCode/
├── packages/                      # Shared packages
│   ├── backend/                  # Core conversion engine
│   │   ├── src/
│   │   │   ├── code.ts          # Main entry point
│   │   │   ├── altNodes/        # Intermediate representation
│   │   │   ├── common/          # Shared utilities
│   │   │   ├── html/            # HTML framework
│   │   │   ├── tailwind/        # Tailwind framework
│   │   │   ├── flutter/         # Flutter framework
│   │   │   ├── swiftui/         # SwiftUI framework
│   │   │   └── compose/         # Compose framework
│   │   └── package.json
│   │
│   ├── plugin-ui/                # Shared UI components
│   │   ├── src/
│   │   │   ├── components/      # React components
│   │   │   ├── App.tsx          # Main UI component
│   │   │   └── index.tsx        # Entry point
│   │   └── package.json
│   │
│   ├── types/                    # TypeScript types
│   ├── eslint-config-custom/     # ESLint config
│   └── tsconfig/                 # TS configs
│
├── apps/                         # Application bundles
│   ├── plugin/                   # Main Figma plugin
│   │   ├── plugin-src/          # Backend entry
│   │   │   └── code.ts          # Compiles to code.js
│   │   ├── ui-src/              # UI entry
│   │   │   └── main.tsx         # Compiles to index.html
│   │   ├── dist/                # Build output
│   │   └── package.json
│   │
│   └── debug/                    # Debug UI (Next.js)
│       └── app/
│
├── manifest.json                 # Figma plugin manifest
├── package.json                  # Root package.json
├── turbo.json                    # Turborepo config
└── pnpm-workspace.yaml           # pnpm workspace config
```

### Where to Make Changes

**Adding/modifying conversion logic**:
- Location: `packages/backend/src/`
- File: Framework-specific files in `{framework}/`

**Adding/modifying UI**:
- Location: `packages/plugin-ui/src/`
- File: Component files in `components/`

**Adding utilities**:
- Location: `packages/backend/src/common/`
- File: Create new utility file

**Adding types**:
- Location: `packages/types/src/`
- File: Create or modify type definition files

## Making Changes

### Example: Fix a Bug in HTML Color Conversion

1. **Identify the file**:
   ```
   packages/backend/src/html/builderImpl/htmlColor.ts
   ```

2. **Make your changes**:
   ```typescript
   // Before
   const hex = rgbToHex(rgb);
   return hex;

   // After
   const hex = rgbToHex(rgb);
   return hex.toLowerCase(); // Fix: always return lowercase
   ```

3. **Save the file**:
   - In dev mode, plugin reloads automatically

4. **Test in Figma**:
   - Select a colored element
   - Check generated code
   - Verify the fix

### Example: Add a New UI Component

1. **Create component file**:
   ```
   packages/plugin-ui/src/components/MyComponent.tsx
   ```

2. **Implement component**:
   ```typescript
   import React from 'react';

   export const MyComponent: React.FC<{ text: string }> = ({ text }) => {
     return <div className="my-component">{text}</div>;
   };
   ```

3. **Use in App**:
   ```typescript
   // packages/plugin-ui/src/App.tsx
   import { MyComponent } from './components/MyComponent';

   export const App = () => {
     return (
       <div>
         <MyComponent text="Hello" />
       </div>
     );
   };
   ```

4. **Test in debug UI**:
   - Open `http://localhost:3000`
   - See your component rendered

## Testing Your Changes

### Manual Testing Workflow

1. **Create test designs in Figma**:
   - Create various layouts (auto-layout, absolute positioning)
   - Add different properties (colors, borders, shadows)
   - Include text with different styles

2. **Run plugin on test designs**:
   - Select elements
   - Generate code
   - Check output

3. **Verify generated code**:
   - Copy code to target environment
   - Render in browser/app
   - Compare with Figma design

### Testing Checklist

- [ ] Auto-layout conversion works
- [ ] Colors convert correctly
- [ ] Sizes are accurate
- [ ] Borders and radius work
- [ ] Shadows render properly
- [ ] Text styles preserved
- [ ] Nested elements work
- [ ] Rotation handled correctly
- [ ] Warnings show for unsupported features

### Test Different Frameworks

```bash
# Test HTML
# Set framework to HTML in settings → Test

# Test Tailwind
# Set framework to Tailwind → Test

# Test Flutter
# Set framework to Flutter → Test

# etc.
```

## Adding a New Framework

### Step-by-Step Guide

**1. Create Framework Directory**:
```bash
mkdir packages/backend/src/myframework
```

**2. Create Main Entry Point**:
```typescript
// packages/backend/src/myframework/myframeworkMain.ts

import { AltSceneNode } from "../altNodes/alt_api_types";
import { PluginSettings } from "../code";

export function myframeworkMain(
  sceneNode: AltSceneNode[],
  settings: PluginSettings
): string {
  // Convert root nodes
  return sceneNode
    .map(node => myframeworkWidgetGenerator(node, settings))
    .join("\n");
}

function myframeworkWidgetGenerator(
  node: AltSceneNode,
  settings: PluginSettings
): string {
  // Your conversion logic here
  return `/* Generated code for ${node.name} */`;
}
```

**3. Create Builder Classes**:

```typescript
// packages/backend/src/myframework/myframeworkDefaultBuilder.ts

export class MyFrameworkDefaultBuilder {
  constructor(
    private node: AltSceneNode,
    private settings: PluginSettings
  ) {}

  // Method chaining for properties
  positionProperties(): this {
    // Add position-related code
    return this;
  }

  shapeProperties(): this {
    // Add shape-related code
    return this;
  }

  build(): string {
    // Return final code
    return `/* Generated */`;
  }
}
```

```typescript
// packages/backend/src/myframework/myframeworkTextBuilder.ts

export class MyFrameworkTextBuilder {
  constructor(
    private node: AltTextNode,
    private settings: PluginSettings
  ) {}

  build(): string {
    // Return text element code
    return `/* Text element */`;
  }
}
```

**4. Create Property Builders**:

```bash
mkdir packages/backend/src/myframework/builderImpl
```

```typescript
// packages/backend/src/myframework/builderImpl/myframeworkColor.ts

export function myframeworkColor(rgb: RGB): string {
  // Convert RGB to framework-specific format
  const r = Math.round(rgb.r * 255);
  const g = Math.round(rgb.g * 255);
  const b = Math.round(rgb.b * 255);

  return `rgb(${r}, ${g}, ${b})`;
}
```

**5. Add to Conversion Router**:

```typescript
// packages/backend/src/code.ts

import { myframeworkMain } from "./myframework/myframeworkMain";

function convertToCode(
  nodes: AltSceneNode[],
  settings: PluginSettings
): string {
  switch (settings.framework) {
    case "HTML":
      return htmlMain(nodes, settings);
    case "Tailwind":
      return tailwindMain(nodes, settings);
    case "MyFramework":
      return myframeworkMain(nodes, settings);
    // ... other frameworks
    default:
      throw new Error(`Unknown framework: ${settings.framework}`);
  }
}
```

**6. Add to Manifest**:

```json
// manifest.json

{
  "codegenLanguages": [
    // ... existing languages
    {
      "label": "My Framework",
      "value": "myframework"
    }
  ]
}
```

**7. Add to UI Settings**:

```typescript
// packages/plugin-ui/src/components/Settings.tsx

const frameworks = [
  { label: "HTML", value: "HTML" },
  { label: "Tailwind", value: "Tailwind" },
  { label: "My Framework", value: "MyFramework" },
  // ... others
];
```

**8. Test Your Framework**:
- Build the project
- Reload plugin in Figma
- Select "My Framework" in settings
- Test conversion

## Common Tasks

### Adding a New Color Format

**File**: `packages/backend/src/common/color.ts`

```typescript
export function rgbToCustomFormat(rgb: RGB): string {
  // Your custom color format
  return `custom(${rgb.r}, ${rgb.g}, ${rgb.b})`;
}
```

### Adding a New Utility

**File**: `packages/backend/src/common/myUtility.ts`

```typescript
export function myUtility(input: string): string {
  // Your utility logic
  return input.toUpperCase();
}
```

### Modifying Plugin Settings

**1. Add to settings type**:
```typescript
// packages/types/src/pluginSettings.ts

export interface PluginSettings {
  // ... existing settings
  myNewSetting: boolean;
}
```

**2. Add default value**:
```typescript
// packages/backend/src/code.ts

const defaultSettings: PluginSettings = {
  // ... existing defaults
  myNewSetting: false,
};
```

**3. Add to UI**:
```typescript
// packages/plugin-ui/src/components/Settings.tsx

<Checkbox
  checked={settings.myNewSetting}
  onChange={(e) => updateSetting('myNewSetting', e.target.checked)}
  label="My New Setting"
/>
```

### Running Linter

```bash
pnpm lint
```

Fix issues automatically:
```bash
pnpm lint --fix
```

### Running Formatter

```bash
pnpm format
```

This formats all `.ts`, `.tsx`, `.css`, and `.md` files.

## Troubleshooting

### Plugin Doesn't Load in Figma

**Problem**: "Plugin failed to load" error

**Solutions**:
1. Check build output exists:
   ```bash
   ls apps/plugin/dist/code.js
   ls apps/plugin/dist/index.html
   ```

2. Rebuild:
   ```bash
   pnpm build
   ```

3. Check manifest.json paths are correct

4. Try importing manifest again in Figma

### Changes Don't Appear in Plugin

**Problem**: Made changes but plugin still shows old code

**Solutions**:
1. In dev mode, save file again to trigger reload

2. Manually reload plugin in Figma:
   - Close plugin window
   - Run plugin again

3. Restart Figma Desktop App

4. Clear pnpm cache and rebuild:
   ```bash
   pnpm store prune
   pnpm install
   pnpm build
   ```

### TypeScript Errors

**Problem**: Type errors in editor

**Solutions**:
1. Restart TypeScript server (VS Code):
   - Cmd/Ctrl + Shift + P
   - "TypeScript: Restart TS Server"

2. Check `tsconfig.json` includes your file

3. Install missing type definitions:
   ```bash
   pnpm add -D @types/package-name
   ```

### Build Fails

**Problem**: `pnpm build` fails

**Solutions**:
1. Check error message for specific issue

2. Clean and rebuild:
   ```bash
   rm -rf node_modules
   rm -rf apps/plugin/dist
   rm -rf packages/*/dist
   pnpm install
   pnpm build
   ```

3. Check Node.js version:
   ```bash
   node --version  # Should be 18+
   ```

4. Update dependencies:
   ```bash
   pnpm update
   ```

### Hot Reload Not Working

**Problem**: UI changes don't appear immediately

**Solutions**:
1. Check dev server is running:
   ```bash
   pnpm dev
   ```

2. Check console for errors

3. Hard reload browser (if using debug UI):
   - Cmd/Ctrl + Shift + R

4. Restart dev server:
   - Stop with Ctrl+C
   - Run `pnpm dev` again

## Best Practices

### Code Style

- Use TypeScript strict mode
- Avoid `any` types
- Use meaningful variable names
- Add JSDoc comments for public APIs
- Follow existing code patterns

### Git Workflow

1. Create feature branch:
   ```bash
   git checkout -b feature/my-feature
   ```

2. Make changes and commit:
   ```bash
   git add .
   git commit -m "Add my feature"
   ```

3. Push and create PR:
   ```bash
   git push origin feature/my-feature
   ```

### Testing

- Test with various Figma designs
- Test all supported frameworks
- Check edge cases (rotation, nesting, etc.)
- Verify warnings for unsupported features

### Documentation

- Update README if adding major features
- Add code comments for complex logic
- Update this guide if changing project structure

---

You're now ready to contribute to FigmaToCode! If you have questions, check the [main README](../README.md) or open an issue on GitHub.
