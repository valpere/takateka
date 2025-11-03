# CLAUDE.md - TakaTeka (ТакаТека)

This file provides guidance to Claude Code when working with the TakaTeka project.

## Project Overview

**TakaTeka** is a library management system that orchestrates document operations using YakaTeka as its document processing engine. It handles library-wide operations, organization, classification, and metadata management.

**Repository**: https://github.com/valpere/takateka
**Project Path**: `~/wrk/projects/library/takateka`
**Shared Context**: `~/wrk/projects/library/context-common` (integration guides)
**Project Context**: `~/wrk/projects/library/context-takateka` (TakaTeka-specific)

## Context Management

**IMPORTANT**: This project uses **external context directories** that are NOT part of the repository:

### Shared Context (`~/wrk/projects/library/context-common/`)
Integration guides for OCR, document converters, image/audio/video processing, AI model runners.

**READ ONLY MODE**: Context files are reference documentation only. Never modify them.

**Key reference files**:
- `00-context.md` - Context index
- `08-golang_all_integration_cheat_sheet.md` - Quick reference for all integrations
- `13-golang_all_integration-06.md` - **Comprehensive integration guide** (primary reference)

### TakaTeka-Specific Context (`~/wrk/projects/library/context-takateka/`)
Project-specific requirements, decisions, and implementation details.

**READ ONLY MODE**: Context files are reference documentation only. Never modify them.

**When making architectural decisions, ALWAYS consult context files first.**

## Project Scope

### What TakaTeka Does
- **Library management**: Add, modify, delete, move, copy documents
- **Organization**: Maintain 3-level category hierarchy
- **Metadata database**: Track all library documents and their metadata
- **Batch operations**: Process multiple documents efficiently
- **Search and discovery**: Find documents by metadata, content, duplicates
- **Classification**: Automatic and manual document categorization
- **Integration**: Communicate with YakaTeka for document processing
- **Reporting**: Library statistics, health checks, integrity verification

### What TakaTeka Does NOT Do
- **Document conversion**: Delegated to YakaTeka
- **OCR processing**: Delegated to YakaTeka
- **Text extraction**: Delegated to YakaTeka
- **Low-level document operations**: YakaTeka's responsibility

### Design Philosophy
- **Librarian pattern**: Orchestrates, doesn't process
- **Separation of concerns**: TakaTeka manages, YakaTeka processes
- **Database-centric**: All metadata in structured storage
- **Batch-first**: Optimize for multi-document operations
- **Integration-ready**: Clean API for YakaTeka communication
- **Extensible**: Plugin architecture for future features

## Relationship with YakaTeka

TakaTeka is the **librarian**, YakaTeka is the **document processor**.

**Communication Evolution**:
1. **Phase 1** (Future): CLI integration - TakaTeka calls YakaTeka CLI via `os/exec`
2. **Phase 2** (Future): REST API - TakaTeka calls YakaTeka HTTP endpoints
3. **Phase 3** (Future): gRPC - Optimized binary protocol for performance

**Current Status**: TakaTeka is not yet started. YakaTeka development comes first.

**Interface Contract** (to be defined):
```
TakaTeka needs:
- Convert document to format X
- Extract text from document
- Parse document metadata
- Analyze document content

YakaTeka provides:
- Conversion service
- Extraction service
- Parsing service
- Analysis service
```

## Technology Stack

### Core Technologies
- **Language**: Go 1.24+
- **CLI Framework**: [Cobra](https://github.com/spf13/cobra) for command structure
- **Configuration**: [Viper](https://github.com/spf13/viper) for hierarchical config
- **Database**: SQLite (default) or PostgreSQL (advanced)
- **Messaging**: Protocol Buffers for gRPC with YakaTeka (future)
- **API Framework**: Gin or Echo for REST API (future)

### Database Schema (Draft)

```sql
-- Documents registry
CREATE TABLE documents (
    id INTEGER PRIMARY KEY,
    path TEXT UNIQUE NOT NULL,
    filename TEXT NOT NULL,
    extension TEXT NOT NULL,
    size_bytes INTEGER NOT NULL,
    checksum_sha256 TEXT NOT NULL,
    category_level1 TEXT,
    category_level2 TEXT,
    category_level3 TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    last_scanned TIMESTAMP
);

-- Metadata extracted from documents
CREATE TABLE metadata (
    id INTEGER PRIMARY KEY,
    document_id INTEGER NOT NULL,
    key TEXT NOT NULL,
    value TEXT,
    source TEXT, -- 'extracted', 'manual', 'yakateka'
    FOREIGN KEY (document_id) REFERENCES documents(id),
    UNIQUE(document_id, key)
);

-- Processing history
CREATE TABLE processing_log (
    id INTEGER PRIMARY KEY,
    document_id INTEGER NOT NULL,
    operation TEXT NOT NULL, -- 'convert', 'parse', 'ocr', etc
    status TEXT NOT NULL, -- 'success', 'failed', 'pending'
    started_at TIMESTAMP,
    completed_at TIMESTAMP,
    error_message TEXT,
    FOREIGN KEY (document_id) REFERENCES documents(id)
);

-- Duplicate groups
CREATE TABLE duplicate_groups (
    id INTEGER PRIMARY KEY,
    checksum TEXT NOT NULL,
    file_count INTEGER NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Categories structure
CREATE TABLE categories (
    id INTEGER PRIMARY KEY,
    level INTEGER NOT NULL, -- 1, 2, or 3
    parent_id INTEGER,
    name TEXT NOT NULL,
    description TEXT,
    FOREIGN KEY (parent_id) REFERENCES categories(id)
);
```

## Project Structure (Future)

```
takateka/
├── cmd/
│   ├── root.go              # Root command setup
│   ├── add.go               # Add documents to library
│   ├── remove.go            # Remove documents
│   ├── move.go              # Move/reorganize documents
│   ├── search.go            # Search operations
│   ├── stats.go             # Library statistics
│   ├── duplicates.go        # Find/manage duplicates
│   ├── classify.go          # Classification operations
│   └── verify.go            # Integrity verification
├── internal/
│   ├── db/                  # Database operations
│   │   ├── sqlite.go
│   │   ├── postgres.go
│   │   ├── migrations/
│   │   └── types.go
│   ├── librarian/           # Core library logic
│   │   ├── scanner.go       # File system scanning
│   │   ├── organizer.go     # Document organization
│   │   ├── classifier.go    # Classification logic
│   │   └── types.go
│   ├── yakateka/            # YakaTeka client
│   │   ├── cli_client.go    # CLI integration
│   │   ├── rest_client.go   # REST API client (future)
│   │   ├── grpc_client.go   # gRPC client (future)
│   │   └── types.go
│   ├── metadata/            # Metadata management
│   │   ├── extractor.go
│   │   ├── merger.go
│   │   └── types.go
│   ├── search/              # Search engine
│   │   ├── indexer.go
│   │   ├── query.go
│   │   └── types.go
│   └── types.go             # Common types
├── pkg/                     # Public API (for future use)
│   └── takateka/
│       └── client.go
├── proto/                   # Protocol Buffers (future gRPC)
│   └── takateka.proto
├── migrations/              # Database migrations
│   ├── 001_initial.sql
│   └── 002_categories.sql
├── config/
│   └── config.yaml          # Default configuration
├── .claude/
│   └── CLAUDE.md            # Symlink to this file
├── go.mod
├── go.sum
├── main.go
├── Makefile
└── README.md
```

## Design Principles

Follow the principles defined in `~/.claude/CLAUDE.md`:

### Core Principles
1. **DRY**: Reusable library management components
2. **YAGNI**: Implement only needed features
3. **KISS**: Simple APIs, clear interfaces
4. **Encapsulation**: Database operations hidden behind interfaces
5. **PoLA**: Intuitive command structure

### SOLID + GRASP
- **Single Responsibility**: Each service handles one aspect (DB, YakaTeka client, etc)
- **Open/Closed**: Easy to add new operations without modifying core
- **Dependency Inversion**: Depend on interfaces (Database interface, YakaTeka interface)
- **Information Expert**: Librarian knows library structure
- **High Cohesion / Low Coupling**: Minimal dependencies between packages

## CLI Command Structure (Future)

```bash
# Library initialization
takateka init [path] [--db=sqlite|postgres]
takateka scan [path] [--recursive] [--update-existing]

# Document operations
takateka add <file|directory> [--category=cat1/cat2/cat3] [--auto-classify]
takateka remove <file|pattern> [--dry-run]
takateka move <source> <dest> [--update-db]
takateka copy <source> <dest>

# Search and discovery
takateka search <query> [--author=name] [--category=path] [--format=pdf]
takateka list [--category=path] [--format=json|table]
takateka duplicates [--by=checksum|content] [--action=list|remove]

# Metadata operations
takateka meta get <file> [--format=json|yaml]
takateka meta set <file> --key=value [--key=value...]
takateka meta sync [--source=yakateka]

# Classification
takateka classify <file|directory> [--auto] [--model=ollama]
takateka categories list [--level=1|2|3]
takateka categories add <name> [--parent=category] [--level=N]

# Batch operations with YakaTeka
takateka convert-all [--from=pdf] [--to=txt] [--parallel=5]
takateka extract-text-all [--ocr] [--parallel=5]
takateka parse-all [--update-db]

# Statistics and health
takateka stats [--by=category|format|size]
takateka verify [--checksums] [--missing] [--duplicates]
takateka report [--format=html|json] [--output=file]
```

## Development Roadmap

### Phase 1: Foundation (Future - After YakaTeka Phase 3)
- [ ] Initialize project structure
- [ ] Setup Cobra CLI framework
- [ ] Setup Viper configuration
- [ ] SQLite database integration
- [ ] Basic migrations system
- [ ] File system scanner

### Phase 2: Core Library Operations (Future)
- [ ] Add documents to library
- [ ] Remove documents
- [ ] Move/reorganize documents
- [ ] Checksum generation and verification
- [ ] Category management
- [ ] Basic search

### Phase 3: YakaTeka CLI Integration (Future)
- [ ] CLI client for YakaTeka
- [ ] Document conversion via YakaTeka
- [ ] Text extraction via YakaTeka
- [ ] Metadata extraction via YakaTeka
- [ ] Error handling and retry logic

### Phase 4: Advanced Features (Future)
- [ ] Duplicate detection (by checksum and content)
- [ ] Auto-classification with rules
- [ ] Advanced search (full-text, fuzzy)
- [ ] Batch operations
- [ ] Library statistics and reporting

### Phase 5: AI-Powered Features (Future)
- [ ] Auto-classification with Ollama/LLMs
- [ ] Content-based similarity search
- [ ] Intelligent metadata extraction
- [ ] Document summarization

### Phase 6: REST API (Future)
- [ ] REST API with Gin/Echo
- [ ] OpenAPI/Swagger documentation
- [ ] Authentication/authorization
- [ ] Rate limiting

### Phase 7: YakaTeka REST Integration (Future)
- [ ] REST client for YakaTeka
- [ ] Replace CLI calls with HTTP
- [ ] Async job queue
- [ ] Progress tracking

### Phase 8: gRPC Integration (Future)
- [ ] Protocol Buffers schema for TakaTeka
- [ ] gRPC server
- [ ] gRPC client for YakaTeka
- [ ] Optimized binary protocol

## Configuration (Future)

```yaml
library:
  path: ~/Documents/library
  scan_on_start: true
  auto_classify: false

database:
  type: sqlite  # or postgres
  path: ~/.takateka/library.db
  # For PostgreSQL:
  # host: localhost
  # port: 5432
  # database: takateka
  # user: takateka
  # password: secret

yakateka:
  mode: cli  # cli, rest, grpc
  cli:
    binary: yakateka
    timeout: 30m
  rest:
    url: http://localhost:8080
    timeout: 5m
  grpc:
    address: localhost:50051
    timeout: 5m

categories:
  levels: 3
  auto_create: false
  validate: true

duplicates:
  strategy: checksum  # checksum, content, both
  action: report  # report, delete, move

classification:
  enabled: false
  engine: rules  # rules, ollama, manual
  ollama:
    model: gemma3:4b
    endpoint: http://localhost:11434

logging:
  level: info
  format: json
```

## Current Status

**TakaTeka is NOT YET STARTED.**

This CLAUDE.md file serves as **architectural documentation and context preservation** for future development.

**Immediate priority**: Develop YakaTeka CLI (Phases 1-3) first.

**When to start TakaTeka**:
1. YakaTeka has stable CLI interface
2. YakaTeka can convert PDF/DOCX to text
3. YakaTeka can extract metadata
4. YakaTeka has basic documentation

## Integration Points with YakaTeka

### Phase 1: CLI Integration (First Implementation)

TakaTeka will shell out to YakaTeka CLI:

```go
// Example: Convert document via YakaTeka CLI
func (c *CLIClient) ConvertDocument(input, output, format string) error {
    cmd := exec.Command("yakateka", "convert", input, output, "--to", format)
    return cmd.Run()
}
```

### Phase 2: REST API Integration

TakaTeka will call YakaTeka HTTP endpoints:

```go
// Example: Convert document via REST API
func (c *RESTClient) ConvertDocument(ctx context.Context, req ConvertRequest) (*ConvertResponse, error) {
    resp, err := c.httpClient.Post("/api/v1/convert", "application/json", req)
    // ...
}
```

### Phase 3: gRPC Integration (Optimized)

TakaTeka will use binary protocol:

```go
// Example: Convert document via gRPC
func (c *GRPCClient) ConvertDocument(ctx context.Context, req *pb.ConvertRequest) (*pb.ConvertResponse, error) {
    return c.client.Convert(ctx, req)
}
```

## Important Notes

- **YakaTeka comes first**: TakaTeka depends on YakaTeka's stable API
- **Context is reference**: External contexts are read-only documentation
- **No migration from old library**: Fresh start with clean architecture
- **Database-centric**: All library state lives in database
- **Separation of concerns**: TakaTeka orchestrates, never processes documents directly

## Questions or Clarifications?

1. **Check context files**: `~/wrk/projects/library/context-common/`
2. **Review this file**: `CLAUDE.md`
3. **Check parent**: `~/.claude/CLAUDE.md`
4. **Check YakaTeka**: `~/wrk/projects/library/yakateka/CLAUDE.md`
5. **Ask the user**: When truly unclear

---

**Created**: 2025-11-03
**Last Updated**: 2025-11-03
**Version**: 1.0.0
**Status**: PLANNING PHASE - Not started yet
