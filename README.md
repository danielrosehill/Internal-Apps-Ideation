# Internal Apps Ideation

This repository demonstrates a **voice-to-implementation workflow** — a process for turning informal voice notes into structured specifications and implementation plans using AI tools.

## The Workflow

```
Voice Note → Transcription (MCP) → Cleanup (Gemini 2.5) → Spec + Plan (Claude Opus 4.5)
```

The complete lineage is preserved in this repository:

1. **Raw voice recording** — An unstructured brain dump about a project idea
2. **Verbatim transcript** — Generated via Gemini Transcription MCP
3. **Cleaned transcript** — Processed with Gemini 2.5 Flash for readability
4. **Formal specification** — Requirements extracted and structured by Claude Opus 4.5
5. **Implementation recommendation** — Stack and architecture suggested by Claude Opus 4.5

## Output

The full workflow output is compiled into a single PDF document:

**[View the PDF: Internal Tools Workspace — From Voice Note to Implementation Plan](output/internal-tools-ideation-flow.pdf)**

## Repository Structure

```
├── input/                  # Source materials
│   ├── SPEC.md            # Formal specification
│   └── turns/             # Transcribed voice memos
├── output/                # Generated outputs
│   ├── internal-tools-ideation-flow.pdf   # Complete workflow document
│   ├── internal-tools-ideation-flow.html  # Source HTML
│   └── turns/             # Intermediate outputs
└── CLAUDE.md              # Project context for Claude Code
```

## The Project Idea

The voice note explores building a unified "AI workspace" — a shell/wrapper that bundles together small AI-powered utilities (whiteboard cleanup, end-of-day logging, etc.) with shared authentication and environment variables. The PDF contains the full ideation journey from raw voice to implementation plan.

## Tools Used

- **Gemini Transcription MCP** — Voice-to-text transcription
- **Gemini 2.5 Flash** — Transcript cleanup and formatting
- **Claude Code (Opus 4.5)** — Specification extraction and implementation planning
- **WeasyPrint** — PDF generation from HTML
