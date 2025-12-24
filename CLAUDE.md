# Internal Apps Ideation

## Purpose

This repository is for **exploring and researching** the concept of a unified internal tools platform for AI-powered productivity workflows.

The goal is to gather ideas, evaluate options, and outline potential approaches for building a modular "AI workspace" that bundles together various personal productivity utilities.

## The Problem Being Explored

Daniel has identified multiple useful AI workflows (whiteboard cleanup, end-of-day logging, voice-to-doc utilities, etc.) but:

- Each is too small to warrant its own standalone app/website
- They share common needs: authentication, API key management, cross-platform access
- There's no clear "glue" framework to bundle these together elegantly

## Key Requirements Under Consideration

1. **Authentication** - Protect API credits (Gemini, OpenRouter, Replicate, etc.)
2. **Cross-platform** - Works on Android phone (for photo uploads) and desktop browser
3. **Modular architecture** - Drop in new utilities as self-contained modules
4. **Shared environment variables** - Common API keys across all modules
5. **Simple navigation** - Sidebar/menu to switch between tools
6. **Framework-agnostic** - Not locked into a specific platform's way of building

## Mobile Context

- **Phone OS**: Android
- Primary mobile use cases: photo capture, voice recording, quick access to utilities

## Workflow Examples Being Considered

- **Daily Whiteboard**: Photo → Gemini vision → cleaned/organized task list → Google Doc
- **End-of-Day Log**: Voice memo → transcription → daily retrospective doc
- Other small AI-powered utilities that aid personal productivity

## Working Style

This is a research and ideation phase. Work with Claude to:

- Research existing frameworks/approaches
- Evaluate deployment options (Vercel, VPS, home server)
- Outline architectural possibilities
- Document findings in this repo

## Deployment Context

Daniel has available:
- VPS
- Vercel
- Home server with Docker services
- N8N for backend workflows

## Directory Structure

- **`/input/`**: Source materials including the formal spec (`SPEC.md`) and transcribed voice memos
- **`/output/`**: Research findings, framework evaluations, and recommendations generated during ideation
