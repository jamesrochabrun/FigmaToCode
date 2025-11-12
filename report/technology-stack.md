# Technology Stack

This document details all technologies, libraries, tools, and dependencies used in FigmaToCode.

## Table of Contents
- [Core Technologies](#core-technologies)
- [Build & Development Tools](#build--development-tools)
- [Frontend Stack](#frontend-stack)
- [Backend Stack](#backend-stack)
- [Code Quality & Testing](#code-quality--testing)
- [Figma Integration](#figma-integration)
- [Dependencies Overview](#dependencies-overview)

## Core Technologies

### Language

**TypeScript 5.8.3**
- Strict mode enabled
- Full type safety across codebase
- Shared type definitions in `packages/types`
- No `any` types in production code (as much as possible)

**Configuration**:
```json
{
  "compilerOptions": {
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "resolveJsonModule": true,
    "moduleResolution": "node"
  }
}
```

### Package Manager

**pnpm 9.14.4**
- Faster than npm/yarn
- Disk space efficient
- Strict dependency resolution
- Workspace support for monorepo

**Workspace Configuration** (`pnpm-workspace.yaml`):
```yaml
packages:
  - 'packages/*'
  - 'apps/*'
```

## Build & Development Tools

### Monorepo Management

**Turborepo 2.5.4**
- Manages builds across packages
- Caches build outputs
- Parallel task execution
- Dependency-aware build order

**Configuration** (`turbo.json`):
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
    },
    "lint": {
      "outputs": []
    }
  }
}
```

**Benefits**:
- Fast incremental builds
- Intelligent caching
- Parallel processing
- Clear dependency graph

### Bundler

**esbuild**
- Extremely fast bundling (10-100x faster than Webpack)
- TypeScript compilation
- Tree shaking
- Minification
- Source maps

**Build Targets**:
- Plugin backend: ES2020, for Figma sandbox
- Plugin UI: ES2020, for modern browsers
- Debug app: Next.js built-in bundler

**Configuration** (from `apps/plugin/esbuild.config.js`):
```javascript
{
  entryPoints: ['plugin-src/code.ts'],
  bundle: true,
  outfile: 'dist/code.js',
  platform: 'node',
  target: 'es2020',
  minify: process.env.NODE_ENV === 'production',
  sourcemap: process.env.NODE_ENV === 'development'
}
```

## Frontend Stack

### UI Framework

**React 19.0.0**
- Modern React with Hooks
- Functional components
- No class components
- Context API for state management

**Key Libraries**:
- `react-dom` - React rendering

### Build Tool

**Vite**
- Fast HMR (Hot Module Replacement)
- Optimized builds
- Native ESM support
- Plugin ecosystem

### Styling

**Tailwind CSS + PostCSS**
- Utility-first CSS
- Custom configuration
- PostCSS for processing
- Autoprefixer for browser compatibility

**Tailwind Config**:
```javascript
{
  content: ['./src/**/*.{js,ts,jsx,tsx}'],
  theme: {
    extend: {
      // Custom colors, spacing, etc.
    }
  }
}
```

### UI Components

**Custom Components** (in `packages/plugin-ui/src/components/`):
- Button
- Input
- Select
- Checkbox
- Modal
- Tabs
- CodeBlock (with syntax highlighting)

**Icon Library**:
- `lucide-react` - Modern icon library

### Utilities

**copy-to-clipboard**
- Client-side clipboard copying
- Cross-browser support
- Simple API

## Backend Stack

### Runtime Environment

**Figma Plugin Sandbox**
- Node-like environment
- Limited global APIs
- No DOM access
- No network access (by design)
- Restricted `require()`

### Core Libraries

**Figma Plugin API**
- `@figma/plugin-typings` - TypeScript types
- Official Figma API
- Version: Latest stable

**Key APIs Used**:
```typescript
// Selection
figma.currentPage.selection

// Export
node.exportAsync({ format: "SVG_STRING" | "JSON_REST_V1" | "PNG" })

// Variables
figma.variables.getVariableByIdAsync(id)

// Storage
figma.clientStorage.setAsync(key, value)
figma.clientStorage.getAsync(key)

// UI Communication
figma.ui.postMessage(message)
figma.on("selectionchange", handler)

// Codegen
figma.codegen.on("generate", handler)
```

### Utilities

**nanoid**
- Unique ID generation
- URL-safe IDs
- Compact and fast

**js-base64**
- Base64 encoding/decoding
- Used for image embedding
- Browser and Node compatible

### Color Processing

**Custom Implementation**
- No external color libraries
- Custom nearest-color algorithm
- RGB/Hex conversion utilities
- Color distance calculation

**Files**:
- `packages/backend/src/common/color.ts`
- `packages/backend/src/common/conversionTables.ts`
- `packages/backend/src/common/nearest-color/`

## Code Quality & Testing

### Linting

**ESLint 9.29.0**
- Custom configuration in `packages/eslint-config-custom`
- Shared across all packages
- TypeScript-aware rules

**Configuration**:
```javascript
module.exports = {
  extends: [
    'eslint:recommended',
    'plugin:@typescript-eslint/recommended'
  ],
  rules: {
    '@typescript-eslint/no-explicit-any': 'warn',
    '@typescript-eslint/explicit-module-boundary-types': 'off'
  }
}
```

### Formatting

**Prettier 3.6.2**
- Opinionated code formatting
- Consistent style across codebase
- Pre-commit hooks (recommended)

**Configuration** (`.prettierrc`):
```json
{
  "semi": true,
  "singleQuote": false,
  "tabWidth": 2,
  "trailingComma": "es5"
}
```

### Testing

**CI/CD**:
- GitHub Actions for CI
- Codecov for coverage tracking
- Automated builds on push

**Test Coverage**:
- Badge in README shows coverage
- Integrated with Codecov

## Figma Integration

### Plugin Manifest

**Version**: Figma Plugin API 1.0.0

**Capabilities**:
```json
{
  "capabilities": ["inspect", "codegen", "vscode"],
  "editorType": ["figma", "dev"],
  "documentAccess": "dynamic-page",
  "networkAccess": {
    "allowedDomains": ["none"]
  }
}
```

### Codegen Languages

Registered in manifest for Dev Mode integration:
```json
{
  "codegenLanguages": [
    { "label": "HTML", "value": "html" },
    { "label": "React (JSX)", "value": "html_jsx" },
    { "label": "Svelte", "value": "html_svelte" },
    { "label": "Styled Components", "value": "html_styled_components" },
    { "label": "Tailwind", "value": "tailwind" },
    { "label": "Tailwind (JSX)", "value": "tailwind_jsx" },
    { "label": "Flutter", "value": "flutter" },
    { "label": "SwiftUI", "value": "swiftUI" }
  ]
}
```

## Dependencies Overview

### Production Dependencies

**Backend** (`packages/backend/package.json`):
```json
{
  "dependencies": {
    "nanoid": "^5.x",
    "js-base64": "^3.x"
  },
  "devDependencies": {
    "@figma/plugin-typings": "^1.x",
    "typescript": "^5.8.3"
  }
}
```

**Plugin UI** (`packages/plugin-ui/package.json`):
```json
{
  "dependencies": {
    "react": "^19.0.0",
    "react-dom": "^19.0.0",
    "lucide-react": "^0.x",
    "copy-to-clipboard": "^3.x"
  },
  "devDependencies": {
    "@types/react": "^19.x",
    "@types/react-dom": "^19.x",
    "vite": "^6.x",
    "tailwindcss": "^3.x",
    "postcss": "^8.x",
    "autoprefixer": "^10.x"
  }
}
```

**Root** (`package.json`):
```json
{
  "devDependencies": {
    "turbo": "^2.5.4",
    "prettier": "^3.6.2",
    "eslint": "^9.29.0",
    "typescript": "^5.8.3"
  }
}
```

### Development Dependencies

**Debug App** (`apps/debug/package.json`):
```json
{
  "dependencies": {
    "next": "^15.x",
    "react": "^19.0.0",
    "react-dom": "^19.0.0"
  }
}
```

## Architecture Patterns

### Design Patterns Used

**Builder Pattern**:
- Each framework has builder classes
- Method chaining for fluent API
- Incremental property accumulation

**Strategy Pattern**:
- Different frameworks implement same conversion interface
- Runtime selection of framework strategy

**Factory Pattern**:
- Builder creation based on node type
- Framework selection based on settings

**Visitor Pattern**:
- Recursive tree traversal
- Process each node with appropriate handler

### Code Organization

**Separation of Concerns**:
```
packages/backend/src/
├── common/           # Shared utilities
├── altNodes/         # Intermediate representation
├── html/             # HTML framework
├── tailwind/         # Tailwind framework
├── flutter/          # Flutter framework
├── swiftui/          # SwiftUI framework
└── compose/          # Compose framework
```

**Each framework follows same structure**:
```
{framework}/
├── {framework}Main.ts              # Entry point
├── {framework}DefaultBuilder.ts    # General builder
├── {framework}TextBuilder.ts       # Text builder
└── builderImpl/                    # Property builders
    ├── {framework}Color.ts
    ├── {framework}Size.ts
    ├── {framework}Border.ts
    ├── {framework}Padding.ts
    └── ...
```

## Performance Optimizations

### Build Performance

**Turborepo Caching**:
- Caches build outputs
- Only rebuilds changed packages
- Shares cache across team

**esbuild Speed**:
- Native Go implementation
- Parallel processing
- Minimal overhead

### Runtime Performance

**Parallel Processing**:
```typescript
await Promise.all([
  processColorVariables(node),
  renderAndAttachSVG(node),
  getStyledTextSegments(node)
]);
```

**Memoization**:
- Variable name caching with `Map`
- Color conversion caching
- Prevent redundant calculations

**Lazy Evaluation**:
- Only process selected nodes
- Don't traverse entire document
- On-demand SVG export

### Performance Monitoring

**Built-in Counters**:
```typescript
let counterGetNodeByIdAsync = 0;
let counterGetStyledTextSegments = 0;
let counterProcessColorVariables = 0;
```

**Performance Marks** (during development):
```typescript
performance.mark("start-conversion");
// ... conversion logic
performance.mark("end-conversion");
performance.measure("conversion", "start-conversion", "end-conversion");
```

## Deployment & Distribution

### Plugin Distribution

**Figma Community**:
- Published to Figma Community
- Plugin ID: 842128343887142055
- Auto-updates for users

### Build Process

**Production Build**:
```bash
pnpm build
```

**Output**:
```
apps/plugin/dist/
├── code.js         # Plugin backend (minified)
└── index.html      # Plugin UI (bundled)
```

**Build Optimization**:
- Minification with esbuild
- Tree shaking (remove unused code)
- No source maps in production
- Optimized bundle size

## Development Workflow

### Local Development

**Start dev mode**:
```bash
pnpm dev
```

**What happens**:
1. Turborepo starts all dev tasks
2. Backend compiles with watch mode
3. UI starts with Vite HMR
4. Debug app starts Next.js server
5. All run concurrently

**Hot Reload**:
- UI changes: Instant HMR
- Backend changes: Fast recompile (~100ms)
- Automatic plugin reload in Figma

### Debug Mode

**Access debug UI**:
```
http://localhost:3000
```

**Features**:
- Browse all UI components
- Test conversion with sample data
- View JSON output
- Compare old vs new conversions

## Version Control

**Git**:
- Repository: GitHub
- Branching: Feature branches
- CI: GitHub Actions

**CI/CD Pipeline** (`.github/workflows/ci.yml`):
```yaml
- Checkout code
- Setup Node.js
- Install pnpm
- Install dependencies
- Run linter
- Run type check
- Build project
- Run tests
- Upload coverage
```

## Browser/Platform Requirements

### Plugin UI Requirements

**Browsers** (for plugin UI iframe):
- Chrome 90+
- Firefox 88+
- Safari 14+
- Edge 90+

**Features Used**:
- ES2020 features
- CSS Flexbox & Grid
- CSS Custom Properties
- backdrop-filter (optional)

### Target Platforms

**Generated Code Targets**:
- **Web**: Modern browsers (ES6+)
- **Flutter**: Flutter 2.0+, Dart 2.12+
- **SwiftUI**: iOS 13+, macOS 10.15+
- **Compose**: Android API 21+, Kotlin 1.5+

## External Services

**None** - The plugin is completely self-contained:
- No external API calls
- No telemetry/analytics
- No network access
- All processing local

**Privacy**:
- No data leaves the user's machine
- No user tracking
- No data collection
- Open source (can be audited)

---

This technology stack provides a modern, performant, and maintainable foundation for FigmaToCode, with emphasis on type safety, fast builds, and excellent developer experience.
