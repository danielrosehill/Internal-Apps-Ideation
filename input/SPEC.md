# Internal Tools Workspace Framework Specification

## Problem Statement

Multiple standalone AI-powered utilities have been (or will be) developed independently. These tools:
- Are too small to warrant individual apps/websites
- Share common needs: auth, API key management, cross-platform access
- Currently lack a unified deployment and access strategy

**What's needed**: A framework/shell that bundles these existing utilities into a cohesive authenticated workspace.

## What This Spec Covers

This is a specification for **the wrapper**, not the tools inside it:
- Deployment framework
- Authentication layer
- Navigation/routing shell
- Shared configuration management
- Module integration pattern

## Core Requirements

### 1. Authentication Layer

- Single login protects all modules
- Prevents unauthorized API credit usage
- Simple to implement (not enterprise-grade)
- Options: OAuth (Google), basic auth, session-based

### 2. Module Shell / Navigation

- Sidebar or menu-based navigation
- Desktop: persistent sidebar
- Mobile (Android): hamburger menu or bottom nav
- Modules render within the shell's content area
- Smooth transitions between modules

### 3. Shared Environment Variables

Centralized configuration for:
- `GEMINI_API_KEY`
- `OPENROUTER_API_KEY`
- `REPLICATE_API_KEY`
- Google API credentials
- Other service keys as needed

Modules inherit these without individual configuration.

### 4. Cross-Platform Support

- **Desktop**: Browser-based
- **Mobile**: Android phone via browser (responsive design)
- Optional: PWA for app-like mobile experience

### 5. Module Integration Pattern

How existing/new tools plug into the workspace:
- Framework-agnostic (React, Vue, vanilla JS modules should all work)
- Self-contained module directories
- Standard interface for mounting into the shell
- Module isolation (one module's failure doesn't crash others)

### 6. Deployment

Available infrastructure:
- **VPS**: Self-hosted option
- **Vercel**: Managed platform with env var sharing
- **Home server**: Docker-based deployment
- **N8N**: Backend workflow orchestration (some modules may call N8N endpoints)

## Non-Requirements

- Not building the AI utilities themselves (already handled)
- Not a no-code platform (Retool, Appsmith, etc.)
- Not a conversational AI interface
- Not a SaaS for external users
- Not tied to a specific AI vendor's ecosystem

## Success Criteria

1. **Integration speed**: New module integrates in <1 hour
2. **Single auth**: One login, all modules accessible
3. **Cross-platform**: Works on desktop browser and Android
4. **Shared config**: Environment variables accessible to all modules without per-module setup
5. **Framework flexibility**: Can drop in React, Vue, or other frontend code
6. **Minimal overhead**: The wrapper adds value, not complexity

## Evaluation Criteria for Framework Selection

1. How easily can custom modules be added?
2. Does it provide auth out of the box or require building it?
3. How does it handle shared environment variables?
4. Mobile/responsive support?
5. Deployment options (Vercel, Docker, VPS)?
6. Lock-in risk - can modules be extracted if framework is abandoned?
7. Learning curve for initial setup?
