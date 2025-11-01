# Roadmap

This document outlines the development roadmap for tust, organized into phases.

## Current Status

- ✅ Project structure established
- ✅ Architecture documented
- 🚧 Initial implementation in progress

## Phase 1: Core Foundation (v0.1.0)

**Goal**: Establish the basic infrastructure and deliver core parameterized testing functionality.

### Milestones

#### 1.1: Project Infrastructure ✅
- [x] Workspace structure
- [x] Crate organization (tust, tust-macros, tust-core, tust-runtime, tust-assertions)
- [x] CI/CD setup (GitHub Actions)
- [x] Documentation structure

#### 1.2: Basic Parsing Pipeline
- [ ] Implement syn-based parsing in `tust-core`
- [ ] Set up four-stage pipeline (Parse → Analyze → Lower → Codegen)
- [ ] Add trybuild for compile-fail tests
- [ ] Create comprehensive error handling

#### 1.3: Parameterized Tests
- [ ] Implement `#[test_case(args...)]` macro
  - [ ] Parse test case attributes
  - [ ] Generate separate test functions for each case
  - [ ] Support custom test names (semicolon syntax)
  - [ ] Validate argument count matches function signature
- [ ] Add `#[test_matrix]` for Cartesian product testing
- [ ] Documentation and examples

#### 1.4: Enhanced Assertions
- [ ] Colored diff output for `assert_eq!`
- [ ] Basic assertion macros:
  - [ ] `assert_ok!`, `assert_err!`
  - [ ] `assert_some!`, `assert_none!`
  - [ ] `assert_matches!` for pattern matching
- [ ] Integration with runtime diff formatting

#### 1.5: Basic Async Support
- [ ] Detect async functions
- [ ] Runtime detection (tokio/async-std via feature flags)
- [ ] Generate appropriate runtime-specific wrappers
- [ ] Error handling for async tests

**Deliverables**:
- Parameterized testing works
- Basic async test support
- Enhanced assertions with colored diffs
- Comprehensive documentation and examples

**Target**: End of Q1 2025

---

## Phase 2: Fixtures and Lifecycle (v0.2.0)

**Goal**: Add fixture dependency injection and lifecycle hooks.

### Milestones

#### 2.1: Basic Fixtures
- [ ] Implement `#[fixture]` attribute
- [ ] Fixture function parsing and validation
- [ ] Inject fixtures into test function parameters
- [ ] Support `#[fixture]` annotation on test parameters
- [ ] Automatic cleanup via Drop trait

#### 2.2: Fixture Scoping
- [ ] Function-scoped fixtures (default)
- [ ] Module-scoped fixtures with `#[fixture(scope = "module")]`
- [ ] Session-scoped fixtures with `#[fixture(scope = "session")]`
- [ ] Lazy fixture evaluation
- [ ] Fixture caching and reuse

#### 2.3: Fixture Dependencies
- [ ] Parse fixture dependencies from parameters
- [ ] Build dependency graph
- [ ] Detect circular dependencies
- [ ] Topological sort for initialization order
- [ ] Nested fixture support

#### 2.4: Lifecycle Hooks
- [ ] `#[before_each]` - runs before each test in module
- [ ] `#[after_each]` - runs after each test
- [ ] `#[before_all]` - runs once before module tests
- [ ] `#[after_all]` - runs once after module tests
- [ ] Proper scoping and execution order

#### 2.5: Parameterized Fixtures
- [ ] Fixtures with `#[test_case]` style parameters
- [ ] Generate fixture variants
- [ ] Combine with parameterized tests

**Deliverables**:
- Full fixture system with dependency injection
- Lifecycle hooks for test setup/teardown
- Scoped fixtures (function/module/session)
- Documentation and migration guide

**Target**: End of Q2 2025

---

## Phase 3: BDD and Organization (v0.3.0)

**Goal**: Add BDD-style organization and rich test output.

### Milestones

#### 3.1: BDD Syntax
- [ ] `#[describe]` for test suites
- [ ] `#[context]` for test contexts
- [ ] `#[it]` for individual tests
- [ ] Nested describe/context blocks
- [ ] Hierarchical test organization

#### 3.2: Improved Test Output
- [ ] Hierarchical test reporting (for BDD)
- [ ] Colored output with status indicators
- [ ] Per-test timing information
- [ ] Summary statistics
- [ ] Progress indicators

#### 3.3: Test Metadata and Filtering
- [ ] `#[tag("name")]` for test categorization
- [ ] Filter tests by tags
- [ ] `#[ignore]` support
- [ ] `#[should_panic]` integration
- [ ] Custom metadata attributes

#### 3.4: Timeout Support
- [ ] `#[test(timeout = "1s")]` attribute
- [ ] Runtime-agnostic timeout implementation
- [ ] Configurable default timeouts
- [ ] Timeout reporting in output

#### 3.5: Serial Execution
- [ ] `#[test(serial)]` for non-parallel tests
- [ ] Serial test groups/pools
- [ ] Integration with cargo test parallelism

**Deliverables**:
- BDD-style test organization (describe/context/it)
- Rich, hierarchical test output
- Test metadata and filtering
- Timeout support
- Serial execution control

**Target**: End of Q3 2025

---

## Phase 4: Advanced Features (v0.4.0)

**Goal**: Add snapshot testing, advanced assertions, and ecosystem integrations.

### Milestones

#### 4.1: Snapshot Testing
- [ ] `assert_snapshot!` macro
- [ ] Inline snapshots with `@` syntax
- [ ] File-based snapshots
- [ ] Snapshot review workflow
- [ ] Update mode for regenerating snapshots
- [ ] Multiple snapshot formats (Debug, Display, JSON, etc.)

#### 4.2: Advanced Assertions
- [ ] Collection matchers:
  - [ ] `assert_contains!`
  - [ ] `assert_all!`
  - [ ] `assert_any!`
- [ ] String matchers:
  - [ ] `assert_str_contains!`
  - [ ] `assert_str_starts_with!`
  - [ ] `assert_regex_match!`
- [ ] Numeric matchers:
  - [ ] `assert_approx_eq!`
  - [ ] `assert_in_range!`
- [ ] Custom matcher trait

#### 4.3: Mocking Integration
- [ ] `#[mock]` attribute for trait mocking
- [ ] Integration with mockall
- [ ] Mock expectations in fixtures
- [ ] Documentation and examples

#### 4.4: File-Based Test Generation
- [ ] `#[test_cases_from_file("path")]` attribute
- [ ] CSV support
- [ ] JSON support
- [ ] Custom parser trait
- [ ] Dynamic test generation

#### 4.5: Custom Test Harness
- [ ] Custom test runner
- [ ] Reporter API and trait
- [ ] JSON output format
- [ ] JUnit XML format
- [ ] TAP format
- [ ] Flaky test detection and retry

**Deliverables**:
- Snapshot testing
- Rich assertion library
- Mocking integration
- File-based test generation
- Custom reporters and output formats

**Target**: End of Q4 2025

---

## Phase 5: Polish and Ecosystem (v1.0.0)

**Goal**: Production-ready release with excellent documentation and tooling.

### Milestones

#### 5.1: Documentation
- [ ] Complete API documentation
- [ ] Tutorial and guides
- [ ] Migration guides from:
  - [ ] Built-in `#[test]`
  - [ ] rstest
  - [ ] test-case
- [ ] Best practices guide
- [ ] FAQ
- [ ] Video tutorials

#### 5.2: Tooling
- [ ] cargo-tust CLI tool
- [ ] Watch mode for continuous testing
- [ ] Coverage integration hooks
- [ ] IDE support (rust-analyzer extensions)
- [ ] Syntax highlighting for common editors

#### 5.3: Performance
- [ ] Optimize compilation time
- [ ] Benchmark macro expansion
- [ ] Minimize runtime overhead
- [ ] Binary size analysis
- [ ] Parallel test execution improvements

#### 5.4: Ecosystem Integration
- [ ] Works with cargo-nextest
- [ ] Integration with popular mocking libraries
- [ ] Property-based testing (proptest) compatibility
- [ ] Compatibility testing with major crates

#### 5.5: Stability
- [ ] Comprehensive test suite (dogfooding)
- [ ] Fuzzing for parser robustness
- [ ] Security audit
- [ ] Performance regression tests
- [ ] SemVer compliance

**Deliverables**:
- Production-ready v1.0.0 release
- Excellent documentation and guides
- Strong tooling support
- Proven stability and performance
- Wide ecosystem compatibility

**Target**: Q2 2026

---

## Post-1.0 Ideas

These are potential features for future consideration:

### Advanced Features
- **Visual Test Reports**: HTML test report generation
- **Test Coverage Visualization**: Built-in coverage reporting
- **Mutation Testing**: Integration with cargo-mutants
- **Benchmark Integration**: Merge with criterion.rs
- **Fuzz Testing**: Integration with cargo-fuzz

### Developer Experience
- **Code Generation**: Generate test scaffolding
- **Auto-fix**: Suggest fixes for common test issues
- **Test Refactoring**: Tools for reorganizing tests

### Integrations
- **CI/CD Templates**: Pre-configured GitHub Actions, GitLab CI, etc.
- **Cloud Test Runners**: Integration with cloud test platforms
- **Test Analytics**: Track test performance over time

---

## Contributing to the Roadmap

This roadmap is a living document. If you have suggestions for features or priorities, please:

1. Open an issue with the `enhancement` label
2. Describe the use case and benefits
3. Reference similar features from other frameworks
4. Discuss with maintainers in the issue thread

---

## Version History

- **v0.1.0** (Target: Q1 2025): Core foundation - parameterized tests, basic async, enhanced assertions
- **v0.2.0** (Target: Q2 2025): Fixtures and lifecycle hooks
- **v0.3.0** (Target: Q3 2025): BDD organization and rich output
- **v0.4.0** (Target: Q4 2025): Snapshot testing, advanced assertions, mocking
- **v1.0.0** (Target: Q2 2026): Production-ready release

---

Last updated: November 2025
