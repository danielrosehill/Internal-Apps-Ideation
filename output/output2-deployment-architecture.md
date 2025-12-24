# Deployment Architecture & PWA Strategy

*Iteration 2 - builds on framework-recommendations.md*

## Low-Code/No-Code: Discarded

Per your feedback, Tooljet/Appsmith/Budibase are off the table. The remaining options are all code-first approaches where you own everything.

---

## Domain Architecture Options

### Option A: Single Domain with Route-Based Modules (Recommended)

```
tools.dsrholdings.cloud/
├── /                    ← Dashboard/home
├── /whiteboard          ← Daily Whiteboard module
├── /eod-log             ← End-of-Day Log module
├── /voice-to-doc        ← Future module
└── /settings            ← API key management, preferences
```

**Pros:**
- Single auth session covers everything
- Single SSL certificate
- Single deployment pipeline
- PWA works seamlessly (one app, multiple views)
- Shared environment variables automatic
- Simpler DNS (one A record or CNAME)

**Cons:**
- All modules must use same framework (React/Vue)
- One broken module could affect others (mitigated by good error boundaries)

**Verdict:** Best fit for your use case. Modules aren't independent services—they're views in a unified workspace.

---

### Option B: Subdomain per Module

```
tools.dsrholdings.cloud/          ← Shell/dashboard
whiteboard.dsrholdings.cloud/     ← Module 1
eod.dsrholdings.cloud/            ← Module 2
```

**Pros:**
- True isolation between modules
- Can use different frameworks per module
- Independent deployments

**Cons:**
- Auth sharing requires extra work (shared cookies with domain scope, or OAuth per subdomain)
- Multiple SSL certs or wildcard cert needed
- Multiple deployment pipelines
- PWA becomes awkward (separate PWA per subdomain, or complex service worker)
- DNS management overhead

**Verdict:** Overkill for internal tools. This pattern suits multi-team enterprise apps, not personal productivity workspace.

---

### Option C: Subdomain for Entire Workspace

```
tools.dsrholdings.cloud/          ← Everything lives here
```

vs.

```
ai.danielrosehill.com/            ← Personal branding alternative
```

**Recommendation:** Use a subdomain of your business domain:
- `tools.dsrholdings.cloud` or `workspace.dsrholdings.cloud`
- Keeps it separate from public-facing sites
- Clear that it's internal tooling

---

## Deployment: Nuts and Bolts

### Vercel Deployment (Simplest Path)

**Setup:**
1. Create Next.js app with App Router
2. Push to GitHub repo
3. Connect repo to Vercel
4. Add custom domain: `tools.dsrholdings.cloud`
5. Set environment variables in Vercel dashboard

**DNS Configuration:**
```
Type: CNAME
Name: tools
Value: cname.vercel-dns.com
```

**Environment Variables (set once in Vercel):**
```
NEXTAUTH_SECRET=<random-string>
NEXTAUTH_URL=https://tools.dsrholdings.cloud
GOOGLE_CLIENT_ID=<oauth-client-id>
GOOGLE_CLIENT_SECRET=<oauth-secret>
GEMINI_API_KEY=<key>
OPENROUTER_API_KEY=<key>
REPLICATE_API_TOKEN=<key>
```

**Deployment Flow:**
```
git push origin main → Vercel auto-builds → Live in ~60 seconds
```

---

### Docker/Home Server Deployment (Alternative)

**For your home server with existing Docker services:**

```dockerfile
FROM node:20-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

FROM node:20-alpine AS runner
WORKDIR /app
ENV NODE_ENV=production
COPY --from=builder /app/.next/standalone ./
COPY --from=builder /app/.next/static ./.next/static
COPY --from=builder /app/public ./public
EXPOSE 3000
CMD ["node", "server.js"]
```

**Docker Compose:**
```yaml
services:
  ai-workspace:
    build: .
    ports:
      - "3100:3000"
    environment:
      - NEXTAUTH_SECRET=${NEXTAUTH_SECRET}
      - NEXTAUTH_URL=https://tools.dsrholdings.cloud
      - GEMINI_API_KEY=${GEMINI_API_KEY}
      # ... other keys
    restart: unless-stopped
```

**Reverse Proxy (Caddy example):**
```
tools.dsrholdings.cloud {
    reverse_proxy localhost:3100
}
```

**Pros of Home Server:**
- No hosting costs
- Full control
- Can integrate with local N8N, Ollama, etc.

**Cons:**
- Requires port forwarding or Cloudflare Tunnel
- You manage uptime
- SSL via Let's Encrypt/Caddy (easy, but still your responsibility)

---

## PWA Strategy for Android

### What PWA Gives You

- **Home screen icon**: Launches like native app
- **No browser chrome**: Full-screen experience
- **Offline caching**: Static assets load instantly
- **Push notifications**: Optional, if needed later

### Implementation in Next.js

**1. Add next-pwa package:**
```bash
npm install next-pwa
```

**2. Configure next.config.js:**
```javascript
const withPWA = require('next-pwa')({
  dest: 'public',
  register: true,
  skipWaiting: true,
  disable: process.env.NODE_ENV === 'development'
})

module.exports = withPWA({
  // your existing next config
})
```

**3. Create public/manifest.json:**
```json
{
  "name": "DSR Tools",
  "short_name": "Tools",
  "description": "AI-powered productivity workspace",
  "start_url": "/",
  "display": "standalone",
  "background_color": "#ffffff",
  "theme_color": "#000000",
  "icons": [
    {
      "src": "/icons/icon-192.png",
      "sizes": "192x192",
      "type": "image/png"
    },
    {
      "src": "/icons/icon-512.png",
      "sizes": "512x512",
      "type": "image/png"
    }
  ]
}
```

**4. Add to layout.tsx head:**
```tsx
<link rel="manifest" href="/manifest.json" />
<meta name="theme-color" content="#000000" />
<link rel="apple-touch-icon" href="/icons/icon-192.png" />
```

### Android Installation Flow

1. User visits `tools.dsrholdings.cloud` in Chrome
2. Chrome prompts "Add to Home Screen" (or user selects from menu)
3. App installs with icon
4. Launches in standalone mode (no URL bar)
5. Auth session persists

### PWA vs Native App

| Aspect | PWA | Native Android App |
|--------|-----|-------------------|
| Installation | Browser prompt | Play Store |
| Development | Web tech (you already know) | Kotlin/Flutter/React Native |
| Updates | Instant (just deploy) | Store review + user update |
| Camera access | ✅ Yes | ✅ Yes |
| File upload | ✅ Yes | ✅ Yes |
| Offline | ✅ With service worker | ✅ Built-in |
| Maintenance | Zero extra | Separate codebase |

**Verdict:** PWA is the right choice. You get app-like experience on Android with zero additional development. Native app would be massive overkill for internal tools.

---

## Recommended Architecture Summary

```
┌─────────────────────────────────────────────────────────┐
│                    DNS                                   │
│  tools.dsrholdings.cloud → Vercel (or home server)      │
└─────────────────────────────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────┐
│                 Next.js Application                      │
│  ┌─────────────────────────────────────────────────────┐│
│  │  NextAuth (Google OAuth)                            ││
│  │  - Single login for entire workspace                ││
│  └─────────────────────────────────────────────────────┘│
│  ┌─────────────────────────────────────────────────────┐│
│  │  Shell Layout                                       ││
│  │  - Sidebar navigation                               ││
│  │  - Responsive (hamburger on mobile)                 ││
│  └─────────────────────────────────────────────────────┘│
│  ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐       │
│  │Whiteboard│ │EOD Log │ │Module 3│ │Module N│       │
│  │  /wb    │ │ /eod   │ │  /...  │ │  /...  │       │
│  └─────────┘ └─────────┘ └─────────┘ └─────────┘       │
│                                                         │
│  Shared: GEMINI_API_KEY, OPENROUTER_API_KEY, etc.      │
└─────────────────────────────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────┐
│                    PWA Layer                             │
│  - manifest.json                                        │
│  - Service worker (via next-pwa)                        │
│  - Installable on Android home screen                   │
└─────────────────────────────────────────────────────────┘
```

---

## Next Steps

1. **Pick domain**: `tools.dsrholdings.cloud` or alternative
2. **Pick deployment**: Vercel (faster) or home server Docker (more control)
3. **Scaffold Next.js + NextAuth + PWA** shell
4. **Test PWA install** on your Android phone
5. **Port first module** into the shell

Want me to proceed with scaffolding the base shell in a new repo?
