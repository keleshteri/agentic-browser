# Repository Guidelines

## Project Structure & Module Organization
src/main hosts the Electron main process bootstrap, window lifecycle, and auto-update wiring. src/preload exposes approved APIs via contextBridge; use this layer for browser-node boundaries. src/renderer is the React UI; keep features in co-located slices with hooks/components separated under feature folders. Static assets live in 
esources. Build artifacts land in out (runtime) and Build (packaging); do not commit either.

## Build, Test, and Development Commands

pm run dev launches the dev server with hot reload across main and renderer. 
pm run start previews the compiled app from out�use it for sanity checks before packaging. 
pm run build runs TypeScript checks then produces optimized bundles. 
pm run lint and 
pm run format enforce style; run 
pm run typecheck if you need to isolate typing issues. Platform-specific packages: 
pm run build:win|mac|linux wrap electron-builder; provide the desired target before merging release changes.

## Coding Style & Naming Conventions
We rely on ESLint flat config plus Prettier. Keep two-space indentation, LF line endings, single quotes, and omit trailing semicolons (Prettier enforces). Name React components and context providers in PascalCase (UserPanel.tsx); hooks and utility modules use camelCase (useUserStore.ts, ipcBridge.ts). Keep preload channel names namespaced (agentic:user-opened). Place styling modules alongside components with the .module.css suffix.

## Testing Guidelines
There is no automated test runner yet, so guard regressions with TypeScript, linting, and manual Electron runs. When adding tests, colocate them with the feature using the *.test.ts(x) or *.spec.ts(x) pattern so future Vitest integration is seamless. Smoke-test critical flows (
pm run dev ? create a window ? trigger IPC) before submitting. Document manual test steps in the PR when you touch startup, IPC, or auto-update logic.

## Commit & Pull Request Guidelines
Follow Conventional Commit prefixes observed in history (feat:, docs:, etc.) with concise, imperative subjects. Squash trivial fixups locally. Each PR should state the problem, the solution summary, verification commands run, and any follow-up work. Link issue IDs when available. Include screenshots or GIFs for renderer changes and note platform coverage for packaging updates.