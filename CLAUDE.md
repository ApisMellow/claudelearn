# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

UIGen is an AI-powered React component generator with live preview. Users describe components in a chat interface, and Claude generates React code that renders in a live preview pane. Components live in a virtual file system (no disk writes). Registered users get project persistence via SQLite.

## Commands

- `npm run setup` — install deps, generate Prisma client, run migrations
- `npm run dev` — start dev server (Next.js + Turbopack, port 3000)
- `npm run build` — production build
- `npm run lint` — ESLint
- `npm run test` — vitest (all tests)
- `npx vitest run src/path/to/test.ts` — run a single test file
- `npm run db:reset` — reset SQLite database

## Architecture

**Two-panel layout**: Chat on the left, Preview/Code on the right. The main layout is in `src/app/main-content.tsx`.

**AI integration**: The `/api/chat` route (`src/app/api/chat/route.ts`) uses the Vercel AI SDK (`ai` package) with `@ai-sdk/anthropic`. The LLM is given two tools — `str_replace_editor` and `file_manager` — to create and modify files in the virtual file system. When no `ANTHROPIC_API_KEY` is set, a `MockLanguageModel` in `src/lib/provider.ts` returns canned responses.

**Virtual File System**: `src/lib/file-system.ts` implements `VirtualFileSystem`, an in-memory tree of `FileNode` objects. It supports create/read/update/delete/rename and serialization for persistence. The LLM tools operate on this VFS server-side, while the client maintains its own instance via `FileSystemProvider` context (`src/lib/contexts/file-system-context.tsx`). Tool calls from the AI stream are replayed client-side via `handleToolCall`.

**Context providers** (both in `src/lib/contexts/`):
- `FileSystemProvider` — owns the client-side VFS instance, handles tool call replay
- `ChatProvider` — wraps Vercel AI SDK's `useChat`, sends serialized VFS state with each request

**Preview**: `src/components/preview/PreviewFrame.tsx` renders generated components. JSX transformation is handled by `src/lib/transform/jsx-transformer.ts` using `@babel/standalone`.

**Auth**: JWT-based auth using `jose` with bcrypt password hashing (`src/lib/auth.ts`). Session stored in cookies.

**Database**: Prisma with SQLite (`prisma/schema.prisma`). Two models: `User` and `Project`. Project stores messages and file system data as JSON strings.

**Server actions**: `src/actions/` contains Next.js server actions for project CRUD.

## Tech Stack

- Next.js 15 (App Router, Turbopack)
- React 19
- TypeScript
- Tailwind CSS v4
- Prisma + SQLite
- Vitest + React Testing Library + jsdom
- shadcn/ui components (in `src/components/ui/`)

The database schema is defined in the @prisma/schema.prisma file. Reference it anytime you need to understand the structure of data stored in the database.
