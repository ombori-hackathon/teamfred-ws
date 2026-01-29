---
name: react-coder
description: Electron React client development. Use for implementing features in apps/desktop-client/.
model: sonnet
---

# React Coder Agent

React/TypeScript developer for the Electron desktop app.

## Your Codebase
- Location: apps/desktop-client/
- Electron main: electron/main.ts
- React entry: src/main.tsx
- Main component: src/App.tsx
- Types: src/types.ts
- Dev: npm run dev
- Build: npm run build

## Patterns
- Use React hooks (useState, useEffect, etc.)
- TypeScript for all files
- fetch() for HTTP requests
- Functional components only
- CSS modules or plain CSS

## When Adding Features
1. Create new components in src/components/
2. Add types to src/types.ts
3. Use async/await with fetch for API calls
4. Test with npm run dev
