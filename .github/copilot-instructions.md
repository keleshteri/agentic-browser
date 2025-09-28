# Agentic Browser - Copilot Instructions

## Project Architecture

This is an **Electron + React + TypeScript** application using modern tooling:
- **electron-vite** for build system (not standard Webpack)
- **Three-process architecture**: Main (`src/main/`), Preload (`src/preload/`), Renderer (`src/renderer/src/`)
- **Context isolation enabled** with `contextBridge` API exposure

## Key File Structure Patterns

```
src/
├── main/index.ts          # Electron main process entry
├── preload/index.ts       # Secure IPC bridge
└── renderer/src/          # React app (note the nested src/)
    ├── App.tsx
    ├── components/
    └── assets/
```

**Critical**: Renderer files are in `src/renderer/src/`, not `src/renderer/` directly.

## Development Workflows

### Essential Commands
- `npm run dev` - Start development with HMR
- `npm run build` - Type check + build all processes
- `npm run typecheck` - Separate type checking for node/web contexts
- `npm run build:win/mac/linux` - Platform-specific distribution builds

### TypeScript Configuration
- **Composite project** with separate configs:
  - `tsconfig.node.json` - Main + Preload processes
  - `tsconfig.web.json` - Renderer (React) process
- **Path alias**: `@renderer/*` maps to `src/renderer/src/*`

## IPC Communication Patterns

### Secure Context Bridge Setup
```typescript
// preload/index.ts - Expose APIs safely
contextBridge.exposeInMainWorld('electron', electronAPI)
contextBridge.exposeInMainWorld('api', api)

// renderer usage
window.electron.ipcRenderer.send('ping')
```

### Current IPC Handlers
- `ping` message in main process (logs 'pong')
- Access to `window.electron.process.versions` for version info

## Build System Specifics

### electron-vite Configuration
- **Renderer alias**: `@renderer` resolves to `src/renderer/src`
- **External deps plugin** for main/preload processes
- **React plugin** only for renderer process

### electron-builder Settings
- **App ID**: `com.electron.app`
- **Multi-platform builds**: Windows (NSIS), macOS (DMG), Linux (AppImage/deb/snap)
- **Auto-updater ready** with `electron-updater` dependency
- **macOS permissions**: Camera, microphone, documents, downloads access pre-configured

## Project-Specific Conventions

### Component Patterns
- **Function components** with explicit return type: `React.JSX.Element`
- **useState hooks** for local state management
- **CSS class-based styling** (no CSS modules or styled-components)

### Security Model
- **Sandbox disabled** (`sandbox: false`) - be aware of security implications
- **Context isolation enabled** - use contextBridge for all IPC
- **External links** open in system browser via `shell.openExternal`

## Asset Handling
- **Static assets** in `src/renderer/src/assets/`
- **App icons** in `build/` and `resources/` directories
- **Vite asset imports** with `?asset` suffix for build resources

## Development Notes
- **Hot reload** works for renderer process only
- **DevTools** accessible via F12 in development
- **Window management** follows macOS conventions (dock activation)
- **Menu bar** auto-hidden by default