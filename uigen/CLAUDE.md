# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

### Development
- `npm run dev` - Start development server with Turbopack
- `npm run dev:daemon` - Run development server in background, logging to logs.txt
- `npm run build` - Build production version
- `npm run start` - Start production server

### Testing and Linting
- `npm test` - Run tests with Vitest
- `npm run lint` - Run Next.js ESLint

### Database
- `npm run setup` - Complete setup (install deps, generate Prisma client, run migrations)
- `npm run db:reset` - Reset database with --force flag

## Architecture

### Core Concept
UIGen is an AI-powered React component generator that uses Claude AI to generate React components in real-time with live preview. The app operates with a **virtual file system** - no files are written to disk during component generation.

### Key Components

**Virtual File System (`src/lib/file-system.ts`)**
- Core abstraction managing in-memory file structure
- Supports standard file operations (create, read, update, delete, rename)
- Serializes/deserializes for persistence and API communication
- Used by AI tools via `str_replace_editor` and `file_manager` tools

**Context Architecture**
- `FileSystemProvider` - Manages virtual file system state and tool calls
- `ChatProvider` - Handles AI chat integration using Vercel AI SDK
- Both contexts work together to enable real-time component generation

**Database Models (Prisma)**
- `User` - Authentication with email/password
- `Project` - Stores chat messages and file system state as JSON
- Uses SQLite with Prisma client generated to `src/generated/prisma`

### AI Integration
- `/api/chat/route.ts` - Main chat endpoint using Anthropic Claude
- Custom tools: `str_replace_editor` and `file_manager` for file manipulation
- System prompt from `src/lib/prompts/generation.tsx`
- Supports fallback mock responses when no Anthropic API key provided

### Frontend Structure
- Next.js 15 with App Router and React 19
- Tailwind CSS v4 with shadcn/ui components
- Monaco Editor for code editing
- React Markdown for chat message rendering
- Resizable panels for chat/editor/preview layout

### Testing
- Vitest with jsdom environment
- React Testing Library for component tests
- Tests located in `__tests__/` directories within each module
- Configuration in `vitest.config.mts`

## Development Notes

### File System Context
The `FileSystemContext` automatically selects `/App.jsx` as the default file when available, or falls back to the first root-level file. This provides a consistent starting point for component generation.

### Project Persistence
Projects are saved only for authenticated users. Anonymous users can work with components but data isn't persisted. The system tracks anonymous work to prompt for account creation.

### AI Tool Integration
The virtual file system integrates with Claude's tool calling via:
- `str_replace_editor`: create, str_replace, insert commands
- `file_manager`: rename, delete commands

Both tools trigger React context updates to keep UI synchronized with file system changes.

### Development Best Practices 
- Use comments sparingly. Only comment complex code.
- The database schema is defined in the @prisma/schema.prisma file. Reference it anytime you need to understand the structure of data stored in the database.