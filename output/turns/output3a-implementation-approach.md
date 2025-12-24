# Implementation Approach

*Iteration 3A - Final recommendation, no alternatives*

## Deployment Decision

**Vercel.** Local deployment discarded.

---

## Stack

- **Framework**: Next.js 14+ (App Router)
- **Auth**: NextAuth.js with Google OAuth
- **Styling**: Tailwind CSS
- **PWA**: next-pwa
- **Deployment**: Vercel
- **Domain**: `tools.dsrholdings.cloud`

---

## Implementation Steps

### 1. Create the Application

```bash
npx create-next-app@latest dsr-tools --typescript --tailwind --app --src-dir
cd dsr-tools
```

### 2. Install Dependencies

```bash
npm install next-auth next-pwa
```

### 3. Configure NextAuth

**`src/app/api/auth/[...nextauth]/route.ts`:**
```typescript
import NextAuth from "next-auth"
import GoogleProvider from "next-auth/providers/google"

const handler = NextAuth({
  providers: [
    GoogleProvider({
      clientId: process.env.GOOGLE_CLIENT_ID!,
      clientSecret: process.env.GOOGLE_CLIENT_SECRET!,
    }),
  ],
  callbacks: {
    async signIn({ user }) {
      // Restrict to your email only
      return user.email === "daniel@danielrosehill.co.il"
    },
  },
})

export { handler as GET, handler as POST }
```

**`src/middleware.ts`:**
```typescript
export { default } from "next-auth/middleware"

export const config = {
  matcher: ["/((?!api/auth|_next|icons|manifest.json).*)"],
}
```

This locks down the entire app. Only you can log in.

### 4. Create Shell Layout

**`src/components/Sidebar.tsx`:**
```typescript
"use client"
import Link from "next/link"
import { usePathname } from "next/navigation"

const modules = [
  { name: "Dashboard", path: "/" },
  { name: "Whiteboard", path: "/whiteboard" },
  { name: "EOD Log", path: "/eod-log" },
]

export function Sidebar() {
  const pathname = usePathname()

  return (
    <nav className="w-64 bg-gray-900 text-white p-4 hidden md:block">
      <h1 className="text-xl font-bold mb-6">DSR Tools</h1>
      <ul className="space-y-2">
        {modules.map((mod) => (
          <li key={mod.path}>
            <Link
              href={mod.path}
              className={`block p-2 rounded ${
                pathname === mod.path ? "bg-gray-700" : "hover:bg-gray-800"
              }`}
            >
              {mod.name}
            </Link>
          </li>
        ))}
      </ul>
    </nav>
  )
}
```

**`src/components/MobileNav.tsx`:**
```typescript
"use client"
import { useState } from "react"
import Link from "next/link"

const modules = [
  { name: "Dashboard", path: "/" },
  { name: "Whiteboard", path: "/whiteboard" },
  { name: "EOD Log", path: "/eod-log" },
]

export function MobileNav() {
  const [open, setOpen] = useState(false)

  return (
    <div className="md:hidden">
      <button
        onClick={() => setOpen(!open)}
        className="p-4 text-2xl"
      >
        ☰
      </button>
      {open && (
        <div className="absolute top-16 left-0 right-0 bg-gray-900 text-white p-4 z-50">
          {modules.map((mod) => (
            <Link
              key={mod.path}
              href={mod.path}
              onClick={() => setOpen(false)}
              className="block p-3 border-b border-gray-700"
            >
              {mod.name}
            </Link>
          ))}
        </div>
      )}
    </div>
  )
}
```

**`src/app/layout.tsx`:**
```typescript
import "./globals.css"
import { Sidebar } from "@/components/Sidebar"
import { MobileNav } from "@/components/MobileNav"
import { getServerSession } from "next-auth"
import SessionProvider from "@/components/SessionProvider"

export default async function RootLayout({
  children,
}: {
  children: React.ReactNode
}) {
  const session = await getServerSession()

  return (
    <html lang="en">
      <head>
        <link rel="manifest" href="/manifest.json" />
        <meta name="theme-color" content="#111827" />
      </head>
      <body>
        <SessionProvider session={session}>
          <div className="flex min-h-screen">
            <Sidebar />
            <div className="flex-1 flex flex-col">
              <MobileNav />
              <main className="flex-1 p-6">{children}</main>
            </div>
          </div>
        </SessionProvider>
      </body>
    </html>
  )
}
```

### 5. Configure PWA

**`next.config.js`:**
```javascript
const withPWA = require("next-pwa")({
  dest: "public",
  register: true,
  skipWaiting: true,
  disable: process.env.NODE_ENV === "development",
})

module.exports = withPWA({
  // next config
})
```

**`public/manifest.json`:**
```json
{
  "name": "DSR Tools",
  "short_name": "Tools",
  "start_url": "/",
  "display": "standalone",
  "background_color": "#111827",
  "theme_color": "#111827",
  "icons": [
    { "src": "/icons/icon-192.png", "sizes": "192x192", "type": "image/png" },
    { "src": "/icons/icon-512.png", "sizes": "512x512", "type": "image/png" }
  ]
}
```

### 6. Create Module Placeholder Pages

**`src/app/page.tsx`:**
```typescript
export default function Dashboard() {
  return (
    <div>
      <h1 className="text-2xl font-bold mb-4">Dashboard</h1>
      <p>Select a tool from the sidebar.</p>
    </div>
  )
}
```

**`src/app/whiteboard/page.tsx`:**
```typescript
export default function Whiteboard() {
  return (
    <div>
      <h1 className="text-2xl font-bold mb-4">Daily Whiteboard</h1>
      {/* Module implementation goes here */}
    </div>
  )
}
```

**`src/app/eod-log/page.tsx`:**
```typescript
export default function EODLog() {
  return (
    <div>
      <h1 className="text-2xl font-bold mb-4">End-of-Day Log</h1>
      {/* Module implementation goes here */}
    </div>
  )
}
```

### 7. Deploy to Vercel

```bash
git init
git add .
git commit -m "Initial shell"
gh repo create dsr-tools --private --push
```

Then in Vercel:
1. Import from GitHub
2. Add environment variables:
   - `NEXTAUTH_SECRET` (generate: `openssl rand -base64 32`)
   - `NEXTAUTH_URL` = `https://tools.dsrholdings.cloud`
   - `GOOGLE_CLIENT_ID`
   - `GOOGLE_CLIENT_SECRET`
   - `GEMINI_API_KEY`
   - `OPENROUTER_API_KEY`
   - `REPLICATE_API_TOKEN`
3. Add custom domain: `tools.dsrholdings.cloud`
4. Deploy

### 8. DNS

Add to your DNS:
```
CNAME  tools  cname.vercel-dns.com
```

### 9. Google OAuth Setup

1. Go to Google Cloud Console → APIs & Services → Credentials
2. Create OAuth 2.0 Client ID
3. Add authorized redirect URI: `https://tools.dsrholdings.cloud/api/auth/callback/google`
4. Copy Client ID and Secret to Vercel env vars

---

## Adding New Modules

To add a new module:

1. Create folder: `src/app/new-module/page.tsx`
2. Add to sidebar array in `Sidebar.tsx` and `MobileNav.tsx`
3. Implement the module
4. Push to GitHub → Vercel auto-deploys

That's it. No configuration, no routing setup, no auth work.

---

## File Structure

```
dsr-tools/
├── src/
│   ├── app/
│   │   ├── api/auth/[...nextauth]/route.ts
│   │   ├── whiteboard/page.tsx
│   │   ├── eod-log/page.tsx
│   │   ├── layout.tsx
│   │   ├── page.tsx
│   │   └── globals.css
│   ├── components/
│   │   ├── Sidebar.tsx
│   │   ├── MobileNav.tsx
│   │   └── SessionProvider.tsx
│   └── middleware.ts
├── public/
│   ├── manifest.json
│   └── icons/
│       ├── icon-192.png
│       └── icon-512.png
├── next.config.js
├── package.json
└── tailwind.config.js
```

---

## Result

- Single authenticated workspace at `tools.dsrholdings.cloud`
- Google login restricted to your email
- Responsive sidebar (desktop) / hamburger (Android)
- PWA installable on Android home screen
- Modules are just folders under `/app`
- All API keys available via `process.env`
- Git push = deployed
