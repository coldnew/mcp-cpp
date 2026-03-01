# AGENTS.md - Developer Guidelines for mcp-cpp

## Build, Lint, and Test Commands

### Quick Reference
```bash
# Build the project
cargo build                    # Debug build
cargo build --release          # Release build

# Format checking
cargo fmt --check              # Check formatting (CI uses this)

# Linting (REQUIRED before commits)
cargo clippy --all-targets --all-features -- -D warnings

# Run all tests
cargo test

# Run a single test
cargo test test_name           # Run tests matching "test_name"
cargo test --test integration # Run specific test file

# Run doc tests
cargo test --doc
```

### E2E Tests (requires Node.js)
```bash
cd test/e2e
npm install
npm test              # Run E2E tests
npm run test:ui      # Run with UI
npm run lint         # Check TypeScript
```

### Python CLI Tool
```bash
cd tools
pip3 install -r requirements.txt
python3 mcp-cli.py --help
python3 mcp-cli.py list-tools
```

---

## Code Style Guidelines

### General Principles
- Write idiomatic Rust - prefer standard library patterns
- Be explicit about error handling - avoid silent failures
- Document public APIs with doc comments (`///` or `//!`)
- Keep functions focused and small (< 100 lines preferred)

### Imports
```rust
// Order: std -> external -> crate
use std::sync::Arc;
use tokio::sync::Mutex;
use serde::{Deserialize, Serialize};
use tracing::{info, instrument};

use crate::module::SubModule;
use crate::error::AppError;
```

### Formatting (enforced by `cargo fmt`)
- 4 space indentation
- Maximum line length: 100 characters
- Use trailing commas in multi-line expressions
- No trailing whitespace

### Naming Conventions
| Element | Convention | Example |
|---------|------------|---------|
| Modules | snake_case | `clangd_session`, `workspace` |
| Structs | PascalCase | `SearchResult`, `ProjectInfo` |
| Enums | PascalCase | `SymbolKind`, `IndexState` |
| Enum variants | PascalCase | `Ok`, `Err`, `Indexing` |
| Functions | snake_case | `get_project_details`, `search_symbols` |
| Variables | snake_case | `build_dir`, `source_files` |
| Constants | SCREAMING_SNAKE | `MAX_RESULTS`, `DEFAULT_TIMEOUT` |
| Traits | PascalCase | `Provider`, `Session` |

### Error Handling
```rust
// Prefer thiserror for custom errors
use thiserror::Error;

#[derive(Error, Debug)]
pub enum AppError {
    #[error("File not found: {0}")]
    FileNotFound(String),
    
    #[error("Connection failed: {0}")]
    ConnectionFailed(String),
}

// Use ? operator with appropriate error types
fn read_file() -> Result<String, AppError> {
    let content = std::fs::read_to_string("file.txt")?;
    Ok(content)
}
```

### Async/Tokio
- Use `tokio::sync` for shared state (Mutex, RwLock)
- Use `#[instrument]` for async functions requiring tracing
- Avoid blocking in async context - use `tokio::task::spawn_blocking`

### Testing
```rust
#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn test_function_name() {
        let result = my_function(42);
        assert_eq!(result, expected_value);
    }

    #[tokio::test]
    async fn test_async_function() {
        let result = async_function().await;
        assert!(result.is_ok());
    }
}
```

### Clippy Rules (strictly enforced)
- **NEVER use `unwrap()`** after checking `is_some()` - use `if let` instead
- **Use `sort_by_key`** with `Reverse` instead of `sort_by` for reverse sorting
- Avoid unnecessary allocations
- Use appropriate collection types

### Module Organization
```rust
// src/module/mod.rs - public API
pub mod sub_module;

pub use sub_module::{PublicType, public_function};

// src/module/sub_module.rs
use crate::other_module::OtherType;

pub struct PublicType { /* ... */ }

pub fn public_function() -> Result<OtherType, Error> { /* ... */ }
```

### Logging
```rust
use tracing::{info, warn, error, instrument};

#[instrument(skip(self), fields(request_id = %id))]
async fn handle_request(&self, id: String) -> Result<Response, Error> {
    info!("Processing request");
    warn!("Fallback triggered for {}", id);
    error!("Failed to connect: {}", err);
}
```

---

## CI/CD Requirements

All PRs must pass:
1. `cargo fmt --check` - formatting
2. `cargo clippy --all-targets --all-features -- -D warnings` - linting
3. `cargo test` - all tests pass
4. `cargo build` - compiles without warnings

---

## File Locations

- **Source**: `src/`
- **Tests**: Inline in source files (`#[cfg(test)]`) + `test/e2e/`
- **Tools**: `tools/` (Python CLI)
- **Config**: `Cargo.toml`, `.github/workflows/`

---

## Key Dependencies

- `rust-mcp-sdk` - MCP protocol
- `lsp-types` - LSP protocol definitions
- `tokio` - Async runtime
- `thiserror` - Error handling
- `tracing` - Logging
- `serde` - Serialization

---

## Important Notes

- Always run `cargo fmt` before committing
- Never commit with clippy warnings
- Use `cargo test` to verify changes
- Test both debug and release builds if performance-related
