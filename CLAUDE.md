# CLAUDE.md - Developer Guide for OpenClaude

> **Essential information for Claude instances working on the OpenClaude codebase**

## Project Overview

OpenClaude is an open-source coding-agent CLI that supports multiple LLM providers (OpenAI, Gemini, DeepSeek, Ollama, GitHub Models, etc.) with a terminal-first workflow. It originated from Claude Code and has been substantially modified to support multiple providers and open use.

### Key Features
- Multi-provider LLM support (OpenAI-compatible APIs, Gemini, GitHub Models, Codex, Ollama, Atomic Chat)
- Terminal-first coding workflows with tools (bash, file operations, grep, glob, agents, tasks, MCP)
- Streaming responses and real-time tool progress
- Provider profiles with guided setup
- VS Code extension integration
- gRPC server mode for headless operation
- MCP (Model Context Protocol) support

## Architecture & Tech Stack

### Core Technologies
- **Language**: TypeScript (ES2022 target, ESNext modules)
- **Runtime**: Node.js ≥20.0.0
- **Package Manager**: Bun (evidenced by `bun.lock`)
- **Build System**: Custom Bun bundler (`scripts/build.ts`)
- **Testing**: Bun's built-in test runner
- **UI Framework**: React + Ink (terminal UI)

### Project Structure
```
src/
├── main.tsx                 # Main entry point
├── Tool.ts                  # Base tool interface
├── Task.ts                  # Task management
├── QueryEngine.ts           # Query processing
├── commands/                # CLI commands
├── tools/                   # Available tools (Bash, File, Grep, etc.)
├── services/                # Core services (API, MCP, analytics, etc.)
├── components/              # React UI components
├── utils/                   # Utilities and helpers
├── grpc/                    # gRPC server implementation
├── proto/                   # Protocol buffer definitions
└── entrypoints/             # Various entry points

vscode-extension/            # VS Code extension
scripts/                     # Build and utility scripts
docs/                        # Documentation
python/                      # Python helpers
bin/                         # CLI launcher
```

### Key Dependencies
- `@anthropic-ai/sdk`: Core Anthropic SDK
- `@modelcontextprotocol/sdk`: MCP support
- `@grpc/grpc-js`: gRPC server functionality
- `react` + `react-reconciler`: Terminal UI via Ink
- `commander`: CLI argument parsing
- `execa`: Process execution
- `chokidar`: File watching
- `chalk`: Terminal colors
- `zod`: Schema validation

## Build System

### Build Process
- **Main build script**: `scripts/build.ts`
- **Bundler**: Bun's native bundler
- **Feature flags**: Configurable at build time for different capabilities
- **Output**: Single distributable JS file in `dist/cli.mjs`
- **Path aliases**: `src/*` mapped in `tsconfig.json`

### Important Build Features
- MACRO.* globals are inlined as build-time constants
- Feature flags control which capabilities are included
- No-telemetry plugin removes tracking code
- Source maps and TypeScript compilation

## Development Workflow

### Setup Commands
```bash
# Install dependencies
bun install

# Build and run
bun run build && node dist/cli.mjs

# Development with auto-rebuild
bun run dev

# Testing
bun test                    # Run all tests
bun run test:coverage       # Generate coverage report
bun run smoke              # Quick smoke test

# Provider-specific development
bun run dev:openai         # OpenAI provider
bun run dev:ollama         # Ollama provider
bun run dev:gemini         # Gemini provider
bun run profile:auto       # Auto-configure provider

# gRPC development
bun run dev:grpc           # Start gRPC server
bun run dev:grpc:cli       # gRPC client
```

### Quality Assurance
```bash
# Type checking
bun run typecheck

# Security scanning
bun run security:pr-scan -- --base origin/main

# Privacy verification
bun run verify:privacy

# System diagnostics
bun run doctor:runtime
```

## Configuration & Environment

### Environment Variables
- **Provider Configuration**: Set in `.env` (copy from `.env.example`)
- **OpenAI**: `CLAUDE_CODE_USE_OPENAI=1`, `OPENAI_API_KEY`, `OPENAI_MODEL`
- **Local models**: `OPENAI_BASE_URL` for Ollama/LM Studio
- **Firecrawl**: `FIRECRAWL_API_KEY` for enhanced web search
- **gRPC**: `GRPC_PORT`, `GRPC_HOST`

### Configuration Files
- **Provider profiles**: `.openclaude-profile.json` (gitignored)
- **Settings**: `~/.claude/settings.json` (agent routing, models)
- **TypeScript**: `tsconfig.json` with strict settings
- **Package**: `package.json` with comprehensive scripts

## Testing Framework

### Test Structure
- **Framework**: Bun's built-in test runner
- **Pattern**: `*.test.ts` files throughout codebase
- **Coverage**: LCOV format with visual heatmap generation
- **Location**: Tests colocated with source code

### Test Categories
- **Unit tests**: Core functionality and utilities
- **Provider tests**: API client testing
- **Integration tests**: Command and workflow testing
- **Security tests**: Intent scanning and validation

### Running Tests
```bash
# Focused test runs
bun run test:provider                    # Provider-specific tests
bun run test:provider-recommendation     # Provider recommendation tests
bun test src/path/to/file.test.ts       # Single file

# Coverage and reporting
open coverage/index.html                 # View coverage report
```

## Key Tools & Services

### Core Tools (src/tools/)
- **BashTool**: Shell command execution
- **FileReadTool**, **FileWriteTool**, **FileEditTool**: File operations
- **GrepTool**, **GlobTool**: Search and pattern matching
- **AgentTool**: Multi-agent workflows
- **TaskCreateTool**, **TaskUpdateTool**: Task management
- **MCPTool**: Model Context Protocol integration
- **WebFetchTool**, **WebSearchTool**: Web operations

### Core Services (src/services/)
- **API services**: Multi-provider LLM clients
- **MCP**: Model Context Protocol implementation
- **Analytics**: Usage tracking and telemetry
- **GitHub**: GitHub integration and workflows
- **OAuth**: Authentication handling

## Development Guidelines

### Code Style
- **Strict TypeScript**: Full type safety required
- **ESLint**: Custom rules enforced
- **Path imports**: Use `src/*` aliases
- **Error handling**: Comprehensive error utilities in `services/api/errors.ts`

### Contributing Workflow
1. **Check existing issues/discussions** before starting
2. **Local setup**: `bun install` → `bun run build` → `bun run dev`
3. **Validation before PR**:
   - `bun run build`
   - `bun run smoke`
   - `bun run test:coverage` (for shared changes)
   - Focused tests for specific changes
4. **Security**: Follow SECURITY.md for security issues

### Provider Development
- **New providers**: Extend API service patterns in `src/services/api/`
- **Agent routing**: Configure in `~/.claude/settings.json`
- **Provider profiles**: Use `/provider` command for setup

## VS Code Extension

Located in `vscode-extension/openclaude-vscode/`:
- Launch integration
- Provider-aware UI
- Theme support
- Control center functionality

## Special Features

### gRPC Server Mode
- **Headless operation**: `npm run dev:grpc`
- **Proto definitions**: `src/proto/openclaude.proto`
- **Bidirectional streaming**: Real-time text and tool calls
- **Test client**: `npm run dev:grpc:cli`

### MCP Integration
- Full Model Context Protocol support
- Server approval workflows
- Resource listing and reading
- Authentication handling

### Multi-Provider Support
- Unified API across providers
- Provider-specific routing
- Cost tracking and limits
- Usage analytics

## Important Notes

### Security
- **No telemetry by default**: Privacy-first design
- **API keys**: Stored in plaintext in settings (keep private)
- **Security scanning**: Built-in intent scanning
- **Verification**: Phone-home prevention checks

### Performance
- **Startup optimization**: Parallel initialization
- **Caching**: LRU caches and prefetching
- **Streaming**: Real-time output and tool progress
- **Memory management**: Careful resource handling

### Debugging
- **System diagnostics**: `bun run doctor:runtime`
- **Coverage reports**: Visual heatmaps
- **Startup profiling**: Built-in performance tracking
- **Error handling**: Comprehensive error utilities

---

**Last Updated**: 2026-04-06  
**Codebase Version**: 0.1.8  
**Node Requirement**: ≥20.0.0