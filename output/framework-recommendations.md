# Framework Recommendations for Internal Tools Workspace

## Summary

After reviewing the requirements, I recommend evaluating these approaches in order of fit:

1. **Next.js + NextAuth** (Recommended for Vercel deployment)
2. **Nuxt 3** (If Vue modules are preferred)
3. **Self-hosted internal tools platform** (Tooljet/Appsmith - with caveats)
4. **Custom micro-frontend shell** (Maximum flexibility, more setup)

---

## Option 1: Next.js + NextAuth (Recommended)

### Why It Fits

- **Native Vercel support**: First-class deployment, env var management built-in
- **NextAuth**: Drop-in authentication with Google OAuth, credentials, etc.
- **App Router**: Modular page structure—each module = a route
- **React-based**: Matches output from Google AI Studio
- **Responsive**: Tailwind or any CSS framework for mobile/desktop

### Module Pattern

```
/app
  /layout.tsx          ← Shell (sidebar, nav, auth wrapper)
  /whiteboard/page.tsx ← Module 1
  /eod-log/page.tsx    ← Module 2
  /new-tool/page.tsx   ← Future modules
```

### Auth Setup

NextAuth provides:
- Google OAuth (one-click login with your Google account)
- Session management
- Route protection via middleware

### Shared Environment Variables

Vercel's env var system:
- Set once in dashboard
- Available to all routes/modules
- Can use `.env.local` for local dev

### Pros

- Minimal learning curve if using React
- Excellent documentation
- Large ecosystem
- Vercel deployment is trivial
- Can also self-host on VPS/Docker

### Cons

- React-only (Vue modules would need adaptation or iframe)
- Some boilerplate for initial setup

### Deployment

- **Vercel**: `git push` → deployed
- **Docker**: Official Next.js Docker support
- **VPS**: Node.js + PM2 or Docker

---

## Option 2: Nuxt 3

### Why Consider It

- If your existing modules are Vue-based
- Similar architecture to Next.js but for Vue
- Built-in auth modules available (nuxt-auth, sidebase/nuxt-auth)

### Module Pattern

```
/pages
  /whiteboard.vue
  /eod-log.vue
/layouts
  /default.vue  ← Shell with sidebar
```

### Pros

- Vue ecosystem
- Good Vercel support
- Auto-imports, clean developer experience

### Cons

- Smaller community than Next.js
- Auth setup slightly more manual than NextAuth

---

## Option 3: Self-Hosted Internal Tools Platform

### Options

- **Tooljet** (open source, self-hostable)
- **Appsmith** (open source, self-hostable)
- **Budibase** (open source)

### Why Consider

- Built-in auth
- Built-in UI components
- Quick to prototype

### Why You Mentioned Avoiding This

Your notes specifically called out not wanting lock-in to "their system to build it." These platforms:
- Use proprietary component systems
- Custom modules require learning their patterns
- Harder to drop in raw React/Vue code

### Verdict

**Skip unless** you want very rapid prototyping and are okay with lock-in. Your existing custom modules won't plug in easily.

---

## Option 4: Custom Micro-Frontend Shell

### What This Means

Build a minimal shell app that:
- Handles auth
- Provides navigation
- Loads modules dynamically (via iframe, module federation, or lazy imports)

### Implementation Approaches

**A. Simple: Iframe-based**
- Shell app with sidebar
- Each module runs independently at its own URL
- Shell embeds modules in iframe
- Auth passed via query param or shared cookie

**B. Advanced: Webpack Module Federation**
- Modules built as "remotes"
- Shell loads them at runtime
- Shared dependencies (React, etc.)
- More complex setup

**C. Monorepo with Shared Shell**
- Turborepo/Nx monorepo
- Shared `packages/shell` component
- Each module imports the shell
- Build as single app or separate deployables

### Pros

- Maximum flexibility
- True framework-agnostic (can mix React, Vue, vanilla)
- No platform lock-in

### Cons

- More upfront engineering
- Auth sharing between modules requires thought
- Deployment more complex

---

## Recommendation Path

Given your constraints and infrastructure:

### For Quick Start (Vercel)

**Next.js + NextAuth**
1. Create Next.js app with App Router
2. Add NextAuth with Google provider
3. Create shell layout with sidebar
4. Each module = a page/folder
5. Deploy to Vercel, set env vars once

### For Home Server (Docker)

Same stack, but:
- Use `next build && next start` in Docker
- Or Coolify/Dokku for PaaS-like experience
- Traefik or Caddy for HTTPS

### For Maximum Flexibility

**Monorepo approach**
- Turborepo with shared shell package
- Each module is a package
- Can deploy together or separately
- More setup, but scales well

---

## Quick Comparison Table

| Aspect | Next.js + NextAuth | Nuxt 3 | Tooljet/Appsmith | Micro-frontend |
|--------|-------------------|--------|------------------|----------------|
| Auth built-in | ✅ (NextAuth) | ⚠️ (needs module) | ✅ | ❌ (build yourself) |
| Module flexibility | React only | Vue only | Platform-specific | Any framework |
| Vercel deploy | ✅ Native | ✅ Good | ❌ Self-host only | ⚠️ Depends |
| Docker deploy | ✅ | ✅ | ✅ | ✅ |
| Learning curve | Low | Low | Low | Medium-High |
| Lock-in risk | Low | Low | High | None |
| Setup time | 1-2 hours | 1-2 hours | 30 min | 4-8 hours |

---

## Next Steps

1. **Decide on framework preference**: React (Next.js) or Vue (Nuxt)?
2. **Prototype the shell**: Create basic app with auth + sidebar
3. **Port one existing module**: Test the integration pattern
4. **Iterate**: Add modules as needed

Would you like me to scaffold a basic Next.js shell with NextAuth as a starting point?
