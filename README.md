# tust

> A modern, batteries-included test framework for Rust with parameterized tests, fixtures, BDD support, and rich assertions

[![Crates.io](https://img.shields.io/crates/v/tust.svg)](https://crates.io/crates/tust)
[![Documentation](https://docs.rs/tust/badge.svg)](https://docs.rs/tust)
[![License](https://img.shields.io/badge/license-MIT%20OR%20Apache--2.0-blue.svg)](LICENSE-MIT)

## Vision

Rust's built-in testing infrastructure is solid but minimal. Many advanced testing features are scattered across separate crates, each solving one specific problem. `tust` aims to provide a comprehensive, batteries-included test framework that brings the best features from testing ecosystems like Jest, RSpec, and pytest to Rust - all in one place.

## Features

- **Parameterized Tests** - Multiple syntaxes for table-driven testing
- **Fixtures** - Dependency injection with automatic cleanup and scoping
- **BDD Support** - Organize tests with describe/context/it blocks
- **Async-First** - Runtime-agnostic async testing without boilerplate
- **Rich Assertions** - Colored diffs, pattern matching, and helpful error messages
- **Lifecycle Hooks** - before/after hooks at multiple scopes
- **Snapshot Testing** - Built-in snapshot assertions
- **Mocking Integration** - Seamless integration with mocking libraries
- **Works with #[test]** - Incrementally adoptable alongside standard tests

## Quick Start

Add `tust` to your dev dependencies:

```toml
[dev-dependencies]
tust = "0.1"
```

Write your first test:

```rust
use tust::prelude::*;

#[test_case(2, 4, 8)]
#[test_case(3, 5, 15)]
#[test_case(0, 10, 0; "zero multiplication")]
fn test_multiply(a: i32, b: i32, expected: i32) {
    assert_eq!(a * b, expected);
}
```

## Example: Fixtures and BDD

```rust
use tust::prelude::*;

#[fixture]
fn database() -> Database {
    Database::connect("test").unwrap()
}

#[describe("User authentication")]
mod authentication {
    use super::*;

    #[before_each]
    fn setup() {
        // Runs before each test
    }

    #[context("with valid credentials")]
    mod valid {
        #[it("should authenticate successfully")]
        fn test_login(#[fixture] database: Database) {
            let result = authenticate(&database, "user", "pass");
            assert_ok!(result);
        }
    }
}
```

## Example: Async Testing

```rust
use tust::prelude::*;

// Runtime-agnostic async test
#[tust::test]
async fn test_async_operation() {
    let result = fetch_data().await;
    assert!(result.is_ok());
}

// With timeout
#[tust::test(timeout = "1s")]
async fn test_with_timeout() {
    slow_operation().await;
}
```

## Why tust?

### Current State of Rust Testing

Rust's testing ecosystem is fragmented. To get modern testing features, you need to combine multiple crates:

- [rstest](https://github.com/la10736/rstest) for fixtures and parameterization
- [test-case](https://github.com/frondeus/test-case) for simple parameterized tests
- [proptest](https://github.com/proptest-rs/proptest) for property-based testing
- [insta](https://github.com/mitsuhiko/insta) for snapshot testing
- [mockall](https://github.com/asomers/mockall) for mocking
- [pretty_assertions](https://github.com/rust-pretty-assertions/rust-pretty-assertions) for colored diffs
- [tokio::test](https://tokio.rs/) or [async-std::test](https://async.rs/) for async
- And more...

Each crate is excellent at what it does, but having them all work together requires configuration and can lead to conflicts or inconsistencies.

### The tust Approach

`tust` consolidates these features into a single, cohesive framework while:

- Maintaining a modular architecture (use only what you need)
- Staying compatible with existing `#[test]` functions
- Working on stable Rust
- Providing excellent error messages with proper spans
- Offering a consistent, ergonomic API across all features

## Feature Comparison

| Feature | Built-in | rstest | test-case | tust |
|---------|----------|--------|-----------|------|
| Parameterized Tests | ❌ | ✅ | ✅ | ✅ |
| Fixtures | ❌ | ✅ | ❌ | ✅ |
| Lifecycle Hooks | ❌ | ❌ | ❌ | ✅ |
| BDD Organization | ❌ | ❌ | ❌ | ✅ |
| Snapshot Testing | ❌ | ❌ | ❌ | ✅ |
| Rich Assertions | ❌ | ❌ | ❌ | ✅ |
| Async Support | ❌ | ⚠️ | ❌ | ✅ |
| Mocking | ❌ | ❌ | ❌ | 🔌 |

Legend: ✅ Built-in, ⚠️ Partial support, ❌ Not supported, 🔌 Integration available

## References and Prior Art

### Name Alternatives

During development, both **must** and **tust** were considered as project names. We chose `tust` for its memorable play on "test" + "rust" and its assertive connotation.

### Inspiration from the Rust Ecosystem

As discussed in this [Reddit thread about Rust's testing tools](https://www.reddit.com/r/rust/comments/wmspjb/why_does_rusts_testing_tools_seem_so_much_less/), Rust's advanced testing capabilities are distributed across many excellent but separate crates:

#### Testing Frameworks
- [**rstest**](https://github.com/la10736/rstest) - Fixture-based test framework with powerful parameterization
- [**test-case**](https://github.com/frondeus/test-case) - Simple attribute macro for parameterized tests
- [**cucumber-rs**](https://github.com/cucumber-rs/cucumber) - BDD framework with Gherkin support

#### Property-Based & Generative Testing
- [**proptest**](https://github.com/proptest-rs/proptest) - Property-based testing with shrinking
- [**quickcheck**](https://github.com/BurntSushi/quickcheck) - QuickCheck-style property testing
- [**arbitrary**](https://github.com/rust-fuzz/arbitrary) - Generate structured test data

#### Snapshot & Assertion Tools
- [**insta**](https://github.com/mitsuhiko/insta) - Snapshot testing with review workflow
- [**pretty_assertions**](https://github.com/rust-pretty-assertions/rust-pretty-assertions) - Colored diff assertions
- [**assert_matches**](https://github.com/murarth/assert_matches) - Pattern matching assertions

#### Mocking
- [**mockall**](https://github.com/asomers/mockall) - Powerful mocking library for Rust
- [**mockito**](https://github.com/lipanski/mockito) - HTTP mocking
- [**wiremock**](https://github.com/LukeMathWalker/wiremock-rs) - HTTP mocking for black-box testing

#### Test Runners & Utilities
- [**cargo-nextest**](https://nexte.st/) - Next-generation test runner with better performance and UX
- [**cargo-watch**](https://github.com/watchexec/cargo-watch) - Watch for changes and run tests
- [**cargo-llvm-cov**](https://github.com/taiki-e/cargo-llvm-cov) - Code coverage with LLVM
- [**tarpaulin**](https://github.com/xd009642/tarpaulin) - Code coverage tool

#### Specialized Testing
- **Automatic Provers** - Formal verification tools like [Kani](https://github.com/model-checking/kani)
- **Design by Contract (DbC)** - Contract-based testing with [contracts](https://gitlab.com/karroffel/contracts)
- **Fuzzing** - [cargo-fuzz](https://github.com/rust-fuzz/cargo-fuzz), [AFL](https://github.com/rust-fuzz/afl.rs)
- **Mutation Testing** - [cargo-mutants](https://github.com/sourcefrog/cargo-mutants)

### Inspiration from Other Languages

`tust` draws inspiration from the best testing frameworks across languages:

- [**Jest**](https://jestjs.io/) (JavaScript) - Snapshot testing, describe/it blocks, rich matchers
- [**RSpec**](https://rspec.info/) (Ruby) - BDD syntax, let blocks (lazy fixtures), custom matchers
- [**pytest**](https://docs.pytest.org/) (Python) - Fixture dependency injection, parameterization
- [**JUnit 5**](https://junit.org/junit5/) (Java) - Lifecycle annotations, nested tests, test templates

## Project Status

`tust` is in early development. The initial focus is on:

1. ✅ Project structure and architecture
2. 🚧 Parameterized tests implementation
3. 🚧 Basic fixtures
4. 📋 Async support
5. 📋 Lifecycle hooks
6. 📋 BDD organization
7. 📋 Rich assertions

See [ROADMAP.md](ROADMAP.md) for detailed implementation phases.

## Contributing

Contributions are welcome! Please see [CONTRIBUTING.md](CONTRIBUTING.md) for guidelines.

## Architecture

For technical details about the implementation, see [ARCHITECTURE.md](ARCHITECTURE.md).

## License

Licensed under either of:

- Apache License, Version 2.0 ([LICENSE-APACHE](LICENSE-APACHE) or http://www.apache.org/licenses/LICENSE-2.0)
- MIT license ([LICENSE-MIT](LICENSE-MIT) or http://opensource.org/licenses/MIT)

at your option.

### Contribution

Unless you explicitly state otherwise, any contribution intentionally submitted for inclusion in the work by you, as defined in the Apache-2.0 license, shall be dual licensed as above, without any additional terms or conditions.
