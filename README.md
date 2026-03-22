# permanager

A CLI tool to scan and list GitHub permanent links embedded in your source code.

## What is this?

When you implement code based on a specification, embedding a GitHub permanent link (a URL pinned to a specific commit SHA) in a comment makes it clear which spec the code is based on:

```rust
// See: https://github.com/owner/repo/blob/abc123def456.../docs/api-spec.md#L15-L30
fn validate_request(req: &Request) -> Result<(), Error> {
    // ...
}
```

`permanager` scans your repository and lists all such links, giving you a complete picture of which parts of your codebase reference which specifications.

## Installation

```bash
cargo install permanager
```

## Usage

### `list` — List all permanent links

```bash
permanager list
```

Scans the current Git repository and prints all GitHub permanent links found in your source files:

```
src/api.rs:12 https://github.com/owner/repo/blob/abc123.../docs/api-spec.md#L15-L30
src/handler.rs:45 https://github.com/owner/repo/blob/abc123.../docs/api-spec.md#L50-L65
src/model.rs:8 https://github.com/owner/repo/blob/def456.../docs/data-model.md#L1-L20
```

## Detection

`permanager` detects URLs matching this pattern:

```
https://github.com/{owner}/{repo}/blob/{40-char SHA}/{path}
https://github.com/{owner}/{repo}/blob/{40-char SHA}/{path}#L{line}
https://github.com/{owner}/{repo}/blob/{40-char SHA}/{path}#L{start}-L{end}
```

- Only full 40-character commit SHAs are recognized — branch names like `main` or `master` are ignored
- All text files are scanned regardless of language or format (`.rs`, `.ts`, `.py`, `.md`, `.yaml`, etc.)
- Files listed in `.gitignore` are excluded

## Why permanent links?

Permanent links create an explicit, machine-readable connection between code and its specification:

- **For humans**: instantly see what spec a piece of code implements, without hunting through docs or chat history
- **For AI**: when reviewing or modifying code, the relevant spec is directly accessible via the link
- **For change tracking**: when a spec changes, you can find all code that references it

## Roadmap

- `--status` flag: check whether each linked spec has changed since the linked commit
- `--fail-on-outdated` flag: exit with non-zero status when outdated links are found (for CI)

## License

MIT
