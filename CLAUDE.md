# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

pdfcpu is a PDF processing library written in Go that supports encryption and offers both an API and a CLI. It supports all PDF versions up to PDF 1.7 (ISO-32000) with basic support for PDF 2.0 (ISO-32000-2).

## Building and Installation

```bash
# Build the CLI from source
cd cmd/pdfcpu
go install

# Verify installation
pdfcpu version
```

## Running Tests

```bash
# Run all tests
go test ./...

# Run tests for a specific package
go test ./pkg/api/test/...

# Run with verbose output
go test -v ./...

# Run tests with coverage (uses custom script)
./coverage.sh

# Run offline coverage tests
./offline_coverage.sh

# Run go vet
go vet -v ./...
```

## Architecture

### Package Structure

- **cmd/pdfcpu**: CLI implementation using the api package
- **pkg/api**: High-level API for integrating pdfcpu into Go backends
  - File-based functions: `CommandFile(inFile, outFile, conf)`
  - Stream-based functions: `Command(rs io.ReadSeeker, w io.Writer, conf)`
  - Each command has both variants
- **pkg/cli**: CLI command handlers
- **pkg/pdfcpu**: Core PDF processing logic
  - **model**: Core types and structures
    - `Context`: Environment for processing PDF files (wraps XRefTable + Configuration)
    - `XRefTable`: Cross-reference table managing PDF objects
    - `Configuration`: Processing configuration including validation mode, encryption, etc.
  - **types**: PDF primitive types (Dict, Array, Name, etc.)
  - **validate**: PDF validation logic
  - **form**: PDF form handling
  - **font**: Font handling
  - **color**: Color handling
  - **create**: PDF creation
  - **primitives**: Building blocks for PDF content
  - **sign**: Digital signature handling
  - **draw**: Drawing operations
  - **matrix**: Transformation matrices
  - **scan**: Content stream scanning
- **pkg/filter**: PDF stream filters (FlateDecode, LZWDecode, ASCII85Decode, etc.)
- **pkg/font**: Font management
- **pkg/log**: Logging utilities
- **pkg/samples**: API usage examples
- **pkg/testdata**: Test resources
- **internal/corefont**: Internal font resources

### Key Concepts

1. **Two API Layers**:
   - File-based layer (used by CLI): Works with file paths
   - Stream-based layer (for backend integration): Works with `io.ReadSeeker`/`io.Writer`

2. **Processing Pipeline**:
   - Read PDF → Parse into Context (with XRefTable) → Validate → Process/Transform → Optimize → Write

3. **Validation Modes** (in `model.Configuration`):
   - `ValidationStrict`: 100% spec compliance (PDF 32000-1:2008)
   - `ValidationRelaxed`: Allows common validation errors found in real-world PDFs

4. **Context Structure**: The `model.Context` type wraps:
   - Configuration (validation mode, encryption settings, etc.)
   - XRefTable (cross-reference table managing all PDF objects)
   - ReadContext (input stream state)
   - OptimizationContext (optimization state)
   - WriteContext (output stream state)

## Error Handling

- This codebase uses `github.com/pkg/errors` for error wrapping
- Error messages should be descriptive but not expose internal details in user-facing contexts
- When adding error handling, use `errors.Wrap()` or `errors.Wrapf()` to add context

## Code Style

- Use `any` instead of `interface{}`
- Follow standard Go conventions
- Use descriptive variable names
- Prefer clarity over brevity

## Testing

- Tests are located in `test` subdirectories within packages (e.g., `pkg/api/test/`)
- Some packages have `*_test.go` files in the same directory
- Test files follow the pattern `*_test.go`
- Use table-driven tests where appropriate
- Tests use testdata from `pkg/testdata/`

## Common Development Workflows

### Adding a New Command

1. Define the command logic in `pkg/pdfcpu/`
2. Add the API functions in `pkg/api/` (both file-based and stream-based variants)
3. Add CLI handlers in `pkg/cli/`
4. Update command mapping in `cmd/pdfcpu/`
5. Add tests in `pkg/api/test/`

### Working with PDF Objects

- All PDF objects extend `types.Object`
- Use `XRefTable` to manage object references
- Dereference indirect references before working with objects
- Be aware of object streams and compressed objects

### Debugging

- Use `-v` flag for verbose output
- Use `-vv` flag for very verbose output (helpful for crash reports)
- Check validation errors with: `pdfcpu validate -vv file.pdf`

## Dependencies

Key external dependencies:
- `github.com/pkg/errors`: Error handling
- `github.com/mattn/go-runewidth`: Text width calculation
- `golang.org/x/image`: Image processing
- `golang.org/x/text`: Text encoding
- `gopkg.in/yaml.v2`: YAML parsing
- `github.com/hhrutter/lzw`: LZW compression
- `github.com/hhrutter/tiff`: TIFF support
- `github.com/hhrutter/pkcs7`: PKCS#7 support for signatures

## CI/CD

- GitHub Actions workflow in `.github/workflows/test.yml`
- Tests run on multiple Go versions (1.23.x, 1.24.x)
- Tests run on multiple platforms (Linux, macOS, Windows, WASM)
- Coverage is tracked via Coveralls
