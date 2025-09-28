# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is an **Electron + React + TypeScript** application called "agentic-browser" using modern tooling and a three-process architecture. The project uses **electron-vite** as the build system (not standard Webpack) and follows Electron security best practices with context isolation.

## Essential Commands

### Development
- `npm run dev` - Start development server with hot module reloading
- `npm start` - Start application in preview mode (production build)

### Building and Type Checking
- `npm run build` - Full build with type checking for all processes
- `npm run typecheck` - Run type checking for both node and web contexts
- `npm run typecheck:node` - Type check main and preload processes only
- `npm run typecheck:web` - Type check renderer (React) process only

### Code Quality
- `npm run lint` - Run ESLint with caching
- `npm run format` - Format code with Prettier

### Distribution Builds
- `npm run build:win` - Build Windows installer (NSIS)
- `npm run build:mac` - Build macOS application (DMG)
- `npm run build:linux` - Build Linux packages (AppImage/deb/snap)
- `npm run build:unpack` - Build without packaging (for testing)

## Architecture

### Three-Process Structure
```
src/
├── main/index.ts          # Electron main process - window management, app lifecycle
├── preload/index.ts       # Security bridge - exposes safe APIs to renderer
└── renderer/src/          # React application (note: nested src directory)
    ├── App.tsx            # Main React component
    ├── components/        # React components
    └── assets/           # Static assets (CSS, images, SVG)
```

**Critical Path Note**: Renderer files are located in `src/renderer/src/`, not `src/renderer/` directly.

### TypeScript Configuration
The project uses a **composite TypeScript setup** with separate configurations:
- `tsconfig.node.json` - Main and Preload processes (Node.js environment)
- `tsconfig.web.json` - Renderer process (React/browser environment)
- `tsconfig.json` - Root configuration referencing both contexts

### Path Aliases
- `@renderer` resolves to `src/renderer/src` (configured in `electron.vite.config.ts`)

## IPC Communication

### Current IPC Setup
- **Ping handler**: `src/main/index.ts:53` - responds to 'ping' messages with console.log('pong')
- **Context bridge**: `src/preload/index.ts` - exposes `window.electron` and `window.api` safely
- **Renderer usage**: `window.electron.ipcRenderer.send('ping')` - as seen in `src/renderer/src/App.tsx:5`

### Security Model
- **Context isolation**: Enabled (default)
- **Sandbox**: Disabled (`sandbox: false` in `src/main/index.ts:16`)
- **External links**: Automatically open in system browser via `shell.openExternal`

### Adding New IPC Handlers
1. Add handler in `src/main/index.ts` using `ipcMain.on()` or `ipcMain.handle()`
2. Expose API through `src/preload/index.ts` contextBridge
3. Use exposed API in renderer components

## Build System (electron-vite)

### Configuration (`electron.vite.config.ts`)
- **Main/Preload**: Uses `externalizeDepsPlugin()` for Node.js dependencies
- **Renderer**: Uses React plugin and `@renderer` path alias
- **Asset handling**: Use `?asset` suffix for build resources (e.g., `icon.png?asset`)

### Development Features
- **Hot reload**: Renderer process only
- **DevTools**: Accessible via F12 in development
- **Environment detection**: Uses `is.dev` from `@electron-toolkit/utils`

## Component Patterns

### React Components
- Use **function components** with explicit `React.JSX.Element` return type
- **IPC communication** via `window.electron.ipcRenderer` in components
- **External links** should use `target="_blank" rel="noreferrer"`

### Styling
- **CSS class-based styling** (no CSS modules or styled-components)
- **Base styles**: `src/renderer/src/assets/base.css` and `main.css`

## Security Considerations

- **Context isolation** prevents direct Node.js access from renderer
- **Preload script** must explicitly expose APIs via `contextBridge`
- **Never expose raw Node.js APIs** to renderer process
- **External URLs** are automatically handled by system browser

## Window Management

- **Default size**: 900x670 pixels
- **Auto-hide menu bar**: Enabled
- **macOS dock behavior**: Re-creates window when dock icon clicked
- **Linux icon**: Handled via build resources

## Build and Distribution

### electron-builder Configuration
- **App ID**: `com.electron.app`
- **Multi-platform support**: Windows, macOS, Linux
- **Auto-updater**: Ready with `electron-updater` dependency
- **macOS permissions**: Pre-configured for camera, microphone, documents, downloads access

### Asset Management
- **App icons**: Located in `build/` and `resources/` directories
- **Vite asset imports**: Use `?asset` suffix for proper bundling
- **Static assets**: Place in `src/renderer/src/assets/`