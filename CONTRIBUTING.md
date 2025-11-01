# Contributing to tust

Thank you for your interest in contributing to tust! This document provides guidelines and information for contributors.

## Getting Started

### Prerequisites

- Rust 1.70 or later
- Git
- Familiarity with procedural macros is helpful but not required

### Setting Up the Development Environment

1. Fork and clone the repository:
   ```bash
   git clone https://github.com/YOUR_USERNAME/tust.git
   cd tust
   ```

2. Build the project:
   ```bash
   cargo build
   ```

3. Run the tests:
   ```bash
   cargo test --all
   ```

4. Check code formatting:
   ```bash
   cargo fmt --all -- --check
   ```

5. Run clippy:
   ```bash
   cargo clippy --all -- -D warnings
   ```

## Project Structure

```
tust/
├── tust/              # Main user-facing API
├── tust-macros/       # Procedural macro definitions
├── tust-core/         # Core parsing and codegen (testable)
├── tust-runtime/      # Runtime support and helpers
└── tust-assertions/   # Assertion macros and matchers
```

### Key Design Principles

1. **Thin Proc Macros**: Keep `tust-macros` thin - it should only convert between `proc_macro` and `proc_macro2` types
2. **Testable Core**: All logic lives in `tust-core` which uses `proc_macro2` and can be unit tested
3. **Pipeline Architecture**: Parse → Analyze → Lower → Codegen
4. **Great Error Messages**: Use proper spans and provide helpful, actionable error messages
5. **Stable Rust**: Everything must work on stable Rust (currently 1.70+)

## How to Contribute

### Reporting Issues

- Check existing issues first to avoid duplicates
- Provide a minimal reproducible example
- Include Rust version and platform information
- Tag appropriately (bug, enhancement, documentation, etc.)

### Suggesting Features

- Open an issue with the `enhancement` label
- Describe the use case and motivation
- Consider how it fits with the existing API
- Reference similar features from other testing frameworks if applicable

### Submitting Pull Requests

1. **Create a feature branch**:
   ```bash
   git checkout -b feature/your-feature-name
   ```

2. **Make your changes**:
   - Follow Rust's standard style guidelines
   - Add tests for new functionality
   - Update documentation as needed
   - Ensure all tests pass: `cargo test --all`
   - Run formatting: `cargo fmt --all`
   - Run clippy: `cargo clippy --all -- -D warnings`

3. **Write good commit messages**:
   - Use present tense ("Add feature" not "Added feature")
   - First line: brief summary (50 chars or less)
   - Blank line, then detailed explanation if needed
   - Reference issue numbers when applicable

4. **Push to your fork**:
   ```bash
   git push origin feature/your-feature-name
   ```

5. **Open a pull request**:
   - Describe what changed and why
   - Link to related issues
   - Include examples of new functionality
   - Be responsive to code review feedback

### Testing Guidelines

#### Unit Tests

Core logic in `tust-core` should have comprehensive unit tests:

```rust
#[cfg(test)]
mod tests {
    use super::*;
    use quote::quote;
    use syn::parse_quote;

    #[test]
    fn test_parse_test_case() {
        let input = parse_quote! {
            #[test_case(1, 2, 3)]
            fn my_test(a: i32, b: i32, c: i32) {}
        };
        let result = parse_test_function(input).unwrap();
        assert_eq!(result.cases.len(), 1);
    }
}
```

#### Compile-Fail Tests

Use `trybuild` for testing error messages:

```rust
#[test]
fn ui_tests() {
    let t = trybuild::TestCases::new();
    t.compile_fail("tests/ui/invalid_*.rs");
    t.pass("tests/ui/valid_*.rs");
}
```

Place test files in `tust-core/tests/ui/`:
- `valid_*.rs` - Should compile successfully
- `invalid_*.rs` - Should fail with specific errors
- `invalid_*.stderr` - Expected error output

#### Integration Tests

Test the full macro expansion:

```rust
#[test]
fn test_parameterized_execution() {
    // Tests that verify generated code actually runs
}
```

### Code Review Process

1. All submissions require review before merging
2. Maintainers will provide feedback within a few days
3. Address review comments and push updates
4. Once approved, a maintainer will merge your PR

### Development Workflow

#### Working on Proc Macros

1. **Use `cargo expand` to inspect generated code**:
   ```bash
   cargo install cargo-expand
   cargo expand --test your_test_name
   ```

2. **Enable debug output during development**:
   ```rust
   eprintln!("Generated: {}", tokens.to_string());
   ```

3. **Test with `trybuild` for error messages**:
   - Add test files to `tests/ui/`
   - Run: `cargo test --test ui`

#### Debugging

- Set `RUST_BACKTRACE=1` for better error traces
- Use `rust-analyzer` in VS Code for macro expansion
- Add `dbg!()` statements in core logic (not in proc macros)

## Coding Standards

### Style

- Follow the [Rust API Guidelines](https://rust-lang.github.io/api-guidelines/)
- Use `cargo fmt` with default settings
- Maximum line length: 100 characters
- Use meaningful variable names
- Add doc comments for all public APIs

### Documentation

- All public items must have doc comments
- Include examples in doc comments where applicable
- Use `///` for item documentation, `//!` for module documentation
- Link to related items with `[item_name]`

Example:
```rust
/// Parses a test case attribute into structured data.
///
/// # Examples
///
/// ```
/// # use tust_core::parse_test_case;
/// let input = /* ... */;
/// let result = parse_test_case(input)?;
/// ```
///
/// # Errors
///
/// Returns an error if the attribute is malformed.
pub fn parse_test_case(input: TokenStream) -> Result<TestCase> {
    // ...
}
```

### Error Handling

- Use `syn::Error` with proper spans for proc macro errors
- Provide actionable error messages
- Include suggestions when possible

Example:
```rust
return Err(syn::Error::new(
    span,
    "expected #[test_case(arg1, arg2)]; note: at least one argument is required"
));
```

## Release Process

(Maintained by project maintainers)

1. Update version in `Cargo.toml`
2. Update CHANGELOG.md
3. Create git tag: `git tag -a v0.x.0 -m "Release v0.x.0"`
4. Push tag: `git push origin v0.x.0`
5. Publish to crates.io: `cargo publish -p tust-core && cargo publish -p tust-macros && cargo publish -p tust`

## Getting Help

- Open an issue with the `question` label
- Check existing documentation and issues
- Ask in discussions for broader questions

## Code of Conduct

Be respectful, inclusive, and professional. We follow the [Rust Code of Conduct](https://www.rust-lang.org/policies/code-of-conduct).

## License

By contributing, you agree that your contributions will be licensed under the same terms as the project (MIT OR Apache-2.0).

Thank you for contributing to tust!
