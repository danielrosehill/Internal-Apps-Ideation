---
title: "Internal Tools Workspace: Ideation Report"
author: "Daniel Rosehill"
date: "24 December 2025"
---

\newpage

# Part 1: Cleaned-Up Specification

## Internal Tools Workspace Framework Specification

### Problem Statement

Multiple standalone AI-powered utilities have been (or will be) developed independently. These tools:

- Are too small to warrant individual apps/websites
- Share common needs: auth, API key management, cross-platform access
- Currently lack a unified deployment and access strategy

**What's needed**: A framework/shell that bundles these existing utilities into a cohesive authenticated workspace.

### What This Spec Covers

This is a specification for **the wrapper**, not the tools inside it:

- Deployment framework
- Authentication layer
- Navigation/routing shell
- Shared configuration management
- Module integration pattern

### Core Requirements

#### 1. Authentication Layer

- Single login protects all modules
- Prevents unauthorized API credit usage
- Simple to implement (not enterprise-grade)
- Method: OAuth (Google)

#### 2. Module Shell / Navigation

- Sidebar or menu-based navigation
- Desktop: persistent sidebar
- Mobile (Android): hamburger menu
- Modules render within the shell's content area
- Smooth transitions between modules

#### 3. Shared Environment Variables

Centralized configuration for:

- `GEMINI_API_KEY`
- `OPENROUTER_API_KEY`
- `REPLICATE_API_KEY`
- Google API credentials
- Other service keys as needed

Modules inherit these without individual configuration.

#### 4. Cross-Platform Support

- **Desktop**: Browser-based
- **Mobile**: Android phone via browser (responsive design)
- **PWA**: Progressive Web App for app-like mobile experience

#### 5. Module Integration Pattern

How existing/new tools plug into the workspace:

- Self-contained module directories
- Standard interface for mounting into the shell
- Module isolation (one module's failure doesn't crash others)

#### 6. Deployment

- **Platform**: Vercel (managed platform with env var sharing)
- **Domain**: Single domain with route-based modules

### Non-Requirements

- Not building the AI utilities themselves (already handled)
- Not a no-code platform (Retool, Appsmith, etc.)
- Not a conversational AI interface
- Not a SaaS for external users
- Not tied to a specific AI vendor's ecosystem
- Not deployed locally/self-hosted

### Success Criteria

1. **Integration speed**: New module integrates in <1 hour
2. **Single auth**: One login, all modules accessible
3. **Cross-platform**: Works on desktop browser and Android
4. **Shared config**: Environment variables accessible to all modules without per-module setup
5. **Minimal overhead**: The wrapper adds value, not complexity

\newpage

# Part 2: Implementation Recommendation

## Stack

- **Framework**: Next.js 14+ (App Router)
- **Auth**: NextAuth.js with Google OAuth
- **Styling**: Tailwind CSS
- **PWA**: next-pwa
- **Deployment**: Vercel
- **Domain**: `tools.dsrholdings.cloud`

## Implementation Steps

### 1. Create the Application

```bash
npx create-next-app@latest dsr-tools --typescript --tailwind --app --src-dir
cd dsr-tools
npm install next-auth next-pwa
```

### 2. Configure NextAuth

Create `src/app/api/auth/[...nextauth]/route.ts`:

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
      return user.email === "daniel@danielrosehill.co.il"
    },
  },
})

export { handler as GET, handler as POST }
```

Create `src/middleware.ts`:

```typescript
export { default } from "next-auth/middleware"

export const config = {
  matcher: ["/((?!api/auth|_next|icons|manifest.json).*)"],
}
```

### 3. Shell Layout with Sidebar

Desktop sidebar and mobile hamburger menu wrap all module content. Modules are simply page folders under `/app`.

### 4. PWA Configuration

Add `next-pwa` wrapper to `next.config.js` and create `public/manifest.json` for Android home screen installation.

### 5. Deploy to Vercel

1. Push to GitHub
2. Import to Vercel
3. Set environment variables (NEXTAUTH_SECRET, API keys)
4. Add custom domain
5. Configure DNS CNAME

## Adding New Modules

1. Create folder: `src/app/new-module/page.tsx`
2. Add to sidebar navigation array
3. Implement the module
4. Push to GitHub → Vercel auto-deploys

## File Structure

```
dsr-tools/
  src/
    app/
      api/auth/[...nextauth]/route.ts
      whiteboard/page.tsx
      eod-log/page.tsx
      layout.tsx
      page.tsx
    components/
      Sidebar.tsx
      MobileNav.tsx
      SessionProvider.tsx
    middleware.ts
  public/
    manifest.json
    icons/
  next.config.js
  package.json
```

## Result

- Single authenticated workspace at `tools.dsrholdings.cloud`
- Google login restricted to your email
- Responsive sidebar (desktop) / hamburger (Android)
- PWA installable on Android home screen
- Modules are just folders under `/app`
- All API keys available via `process.env`
- Git push = deployed

\newpage

# Part 3: Original Voice Note (Verbatim)

*Transcribed: 24 December 2025*

---

Okay, so the purpose of this voice note that I will transcribe, as usual, using Whisper or Gemini is to provide a bit of context about a I guess, workflow work process that I'd really like to roll out properly in the coming year, 2026, and I'll describe sort of what I'm trying to achieve and what I haven't figured out yet.

So with the rapid rise of AI and all that, there's just so many different workflows that are potentially helpful. I'll give an example of one that I just thought about. I was working on a whiteboard, working on my whiteboard. It's one of those days where I feel I need to jot down what I need to get done—mostly very boring stuff filing tax returns. What else do I have up there at the moment? And then a few more interesting things about NFC tags I need to set up. I'm looking over now cleaning. Of course, another scintillating task.

So I have a few things and it really does help me to kind of have those just written out on a whiteboard even if it's just in my office. I can kind of check on it. So here's a workflow that I thought about. I've done a few very effective ones with Nano Banana that have basically involved cleaning up a whiteboard. So you take a whiteboard, you know, it's kind of a bit messy and cleaning it up. The use case for that was sketches for product ideas.

But what I was thinking in this work, in this workflow is I take I was thinking of a name, a catchy name might be something like a Whiteboard a Day or My Daily Whiteboard or something.

And I can clean up the whiteboard as I've done before and that works really well. It's a very effective workflow. Not only that, it would create a Google Doc with my daily whiteboard. So, in other words, the whiteboard or whiteboards, if I might occasionally do a few of them, I take photos on my phone. This is what I this is what I want to do today or even this week for that matter.

The photos go up to Gemini, which uses Nano Banana to clean them up firstly. Secondly, to extract the data. So, basically, a vision workflow kind of like OCR, I guess. Really a good workflow for Gemini because it's multimodal. Maybe do even a little bit of organization because whiteboards show connections. It can infer prioritization, it could sort the whiteboards. Today, for example, I have a mixture of business and personal tasks here. The business ones are the bureaucracy. The personal ones are the the household stuff. If it gave me a list like today's date 24th of December: personal tasks, task, bullet points; business tasks, task, bullet points, and generated a Google Doc in a folder. That would be a great workflow.

## The Problem of Workflow Deployment

Now here's what I'm, here's my question. Over the course of the last year, I found quite a few workflows like this and a few of them have been very, very useful. Another one that I kind of want to roll out for my own sort of just work-life balance, productivity planning, all the rest of it, is an end-of-day log. It's a practice I really want to start doing because I really like the idea of Kaizen or daily incremental improvement, and I tend to sometimes say, "I really want to finish this. I'm going to go for it at all costs. I'm going to have another cup of coffee. I'm going to burn the midnight oil," and it's really bad. It's a habit I want to get out of.

What I realized that the driver for me is usually if I don't if I don't do it this way, I won't remember where I left off, what the project was, blah, blah, blah, and therefore I thought as a sort of hack for my own brain would be to record an end-of-day diary just when I'm about to step away from the workstation for the day and say, "Okay, this is where we're up to. These are the blockers, these are the projects," and then generate, record that as a voice memo maybe, generate a Google Doc from that, and then in the morning, I can look at my retrospective and say, "Okay, I'm ready to pick up."

So these are two ideas of workflows that are intended as I guess little assistance for me in my daily life and my productivity. But neither is so monumental that I want it to be like its own website or its own app. So what I'm getting to is this. There's a lot of technology that's come to market in the last year. I love the Google website builder and they rolled it out an AI Studio. It's easier than ever, especially with vibe coding, to actually say, "I want this. Let's make it. Let's do it." Great. And the only question that I have becomes if I create deployment basically. If I create a bunch of different utilities, they need to be the commonality as as their internal tools is probably authentication. That's only because it's using Gemini API credits. I don't want anyone on the internet to be able to use my whiteboard cleaner and, you know, I get a bill for the whole of the internet using Gemini on my dime.

So I have authentication is a requirement just to protect the, you know, the API and the integrations. The most elegant and the other thing is cross-platform. My whiteboard, for example, that app actually probably makes a lot more sense to use that from my phone where I can just take photos, upload it, hit send, and send it off to the to the queue. The whiteboard app is a good example, actually, I think of an N8N workflow. I can see it being quite fairly straightforward to do that. A form capturing images, send, and then send that off to Gemini and then send, you know, create the Google Docs. So, I've done a lot of that stuff. The back end architecture is pretty straightforward, or you can get or you can vibe code your way to it. So it's purely a problem in my mind of front end in that I haven't really seen and I don't even know like Streamlit's another example. Streamlit and Radio, I've done a lot of tools using those things.

I frequently find myself asking, "Great, we can build all these utilities. They want to be authenticated, but I need I want a workspace. I want like an app that is my AI assistant productivity center. It's not a conversational app. It's bundling together all these standalone utilities. I can just go into my, let's call it Daniel's AI workspace or my company is called Diesser Holdings. It could be Diesser Holdings, I don't know, Internal Productivity Zone." These are terrible names, but that's the idea. In terms of deployment, I have a VPS. I have Vercel. I have a home server. It's not that I'm lacking for places to put stuff. I'm just trying to figure out what's the most sensible way, like what I want to do because these are kind of piecemeal apps. I might have come up with eight of these over the course of the year. So it's not like a gigantic spiral of different things. It's more that I want to have something that I can create a new. I tend to work on these as modules, right? So I might spend an hour today getting this workflow, the whiteboard thing working. Then what I'd like to do is integrate that into my workspace. So it's kind of I'd like to have that kind of wrapper that I can add elements as I wish. And then the other thing that makes a lot of sense to me because these would be my own internal tools, whatever, I know in Vercel, for example, you can share environment variables. The ones I'm always using are Gemini, Open Router, Replicate, you know, probably five or six of them for different AI things. So that makes a lot of sense to me because if I could have this space, I could share the variables.

## The Need for an AI Glue Framework

I guess what I'm looking for, to be more concrete, is there's a million and one great projects on GitHub and the internet and open source, and what I'm looking for is is not the not the flashy tool or framework, it's the it's the glue. It's a framework that's whose the purpose is to bundle together different AI tools. And that isn't something like, let's say, I'm trying to think of an example, Divi. I think it's called Divi. All the name sounds similar after a while where it's kind of you are tied into using their system to build it. No, I'd like to build these things are probably best custom built. When you use Google AI Studio, you tend to get React or Vue frameworks. That's fine for me. I just want the sort of private environment where I have the ability to drop in my own modules, have a, and again, most of the challenge here is actually UI UX or just simple things like that. If I had a workspace on my phone and I have these different modules, the end of day thing I talked about, the whiteboard thing, all you need is a menu basically. can be a sidebar, and then on the desktop it can be another also a menu. You just need that navigation core to wrap around your elements, common authentication and support for the ability to drop in and deploy additional modules. That's pretty much the spec that I have in mind here.
