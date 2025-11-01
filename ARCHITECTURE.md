# Architecture

This document describes the technical architecture and design decisions of the tust test framework.

## Overview

tust is implemented as a multi-crate workspace using procedural macros. The architecture follows a clear separation of concerns to enable testing, maintainability, and extensibility.

## Crate Structure

```
tust/
├── tust/              # Main user-facing crate
├── tust-macros/       # Proc macro wrappers
├── tust-core/         # Core logic (testable)
├── tust-runtime/      # Runtime support
└── tust-assertions/   # Assertion library
```

### tust

**Purpose**: Main entry point for users

**Key Responsibilities**:
- Re-export all macros from `tust-macros`
- Re-export runtime utilities from `tust-runtime`
- Re-export assertions from `tust-assertions`
- Provide `prelude` module for convenient imports
- Define feature flags for optional integrations

**API Surface**:
```rust
// Users write:
use tust::prelude::*;

#[test_case(1, 2, 3)]
fn my_test(a: i32, b: i32, c: i32) {
    assert_eq!(a + b, c);
}
```

### tust-macros

**Purpose**: Thin wrapper around proc macros

**Key Responsibilities**:
- Convert `proc_macro::TokenStream` to `proc_macro2::TokenStream`
- Call into `tust-core` for all logic
- Convert results back to `proc_macro::TokenStream`

**Why This Exists**:
- Proc macro crates can't be unit tested directly
- Keeps compilation fast (minimal logic)
- Allows `tust-core` to be fully testable

**Example**:
```rust
use proc_macro::TokenStream;
use tust_core;

#[proc_macro_attribute]
pub fn test_case(attr: TokenStream, item: TokenStream) -> TokenStream {
    let attr2 = proc_macro2::TokenStream::from(attr);
    let item2 = proc_macro2::TokenStream::from(item);

    match tust_core::expand_test_case(attr2, item2) {
        Ok(tokens) => TokenStream::from(tokens),
        Err(err) => TokenStream::from(err.to_compile_error()),
    }
}
```

### tust-core

**Purpose**: All parsing, analysis, and code generation logic

**Key Responsibilities**:
- Parse macro inputs using `syn`
- Validate inputs and produce helpful errors
- Build intermediate representations
- Generate output code using `quote`
- **Everything here is testable**

**Architecture**: Four-stage pipeline

#### Stage 1: Parse

Convert `TokenStream` into typed AST using `syn`:

```rust
pub fn parse_test_case(attr: TokenStream2, item: TokenStream2) -> Result<ParsedTestCase> {
    let args: TestCaseArgs = syn::parse2(attr)?;
    let func: ItemFn = syn::parse2(item)?;

    Ok(ParsedTestCase {
        args,
        func,
    })
}
```

**Key Types**:
- `TestCaseArgs`: Parsed `#[test_case(...)]` arguments
- `FixtureArgs`: Parsed `#[fixture]` arguments
- `SuiteArgs`: Parsed `#[describe]`, `#[context]` arguments

#### Stage 2: Analyze

Validate and extract semantic information:

```rust
pub fn analyze_test_case(parsed: ParsedTestCase) -> Result<TestCaseModel> {
    // Validate arguments match function signature
    validate_arity(&parsed.args, &parsed.func)?;

    // Extract metadata
    let metadata = extract_metadata(&parsed.func)?;

    // Build fixture dependency graph
    let fixtures = resolve_fixtures(&parsed.func)?;

    Ok(TestCaseModel {
        test_name: parsed.func.sig.ident.clone(),
        cases: parsed.args.cases,
        fixtures,
        metadata,
    })
}
```

**Responsibilities**:
- Type checking and validation
- Fixture dependency resolution
- Metadata extraction (timeout, serial, etc.)
- Semantic error reporting

#### Stage 3: Lower

Transform domain model into intermediate representation:

```rust
pub fn lower_test_case(model: TestCaseModel) -> CodegenIR {
    let test_fns = model.cases.iter().map(|case| {
        TestFnIR {
            name: format_ident!("{}_{}", model.test_name, case.index),
            args: case.args.clone(),
            body: model.body.clone(),
            fixtures: model.fixtures.clone(),
        }
    }).collect();

    CodegenIR { test_fns }
}
```

**Purpose**:
- Prepare for code generation
- Handle name generation
- Structure output organization

#### Stage 4: Codegen

Generate final code using `quote!`:

```rust
pub fn codegen_test_case(ir: CodegenIR) -> TokenStream2 {
    let test_fns = ir.test_fns.iter().map(|test_fn| {
        let name = &test_fn.name;
        let args = &test_fn.args;
        let body = &test_fn.body;

        quote! {
            #[test]
            fn #name() {
                let (#(#args),*) = (#(#args),*);
                #body
            }
        }
    });

    quote! {
        #(#test_fns)*
    }
}
```

**Key Techniques**:
- Preserve original spans for error messages
- Generate unique names for parameterized tests
- Inject fixture setup code
- Handle async transformation

### tust-runtime

**Purpose**: Runtime support and utilities

**Key Responsibilities**:
- Test context management
- Fixture storage and lifecycle
- Colored output formatting
- Diff generation
- Async runtime detection
- Timeout implementation

**Example Components**:

```rust
// Fixture registry
pub struct FixtureRegistry {
    fixtures: HashMap<TypeId, Box<dyn Any>>,
}

impl FixtureRegistry {
    pub fn get_or_create<T: 'static>(&mut self, ctor: impl FnOnce() -> T) -> &T {
        // Lazy fixture initialization
    }
}

// Diff formatting
pub fn format_diff(left: &str, right: &str) -> String {
    // Use `similar` crate for colored diffs
}

// Async runtime detection
pub fn detect_runtime() -> Runtime {
    #[cfg(feature = "tokio")]
    if tokio::runtime::Handle::try_current().is_ok() {
        return Runtime::Tokio;
    }

    #[cfg(feature = "async-std")]
    // ...

    Runtime::None
}
```

### tust-assertions

**Purpose**: Rich assertion macros and matchers

**Key Responsibilities**:
- Enhanced `assert_eq!` with colored diffs
- Pattern matching assertions
- Collection assertions
- String assertions
- Snapshot testing
- Custom matcher trait

**Example**:
```rust
#[macro_export]
macro_rules! assert_eq {
    ($left:expr, $right:expr $(,)?) => {
        match (&$left, &$right) {
            (left_val, right_val) => {
                if !(*left_val == *right_val) {
                    let diff = $crate::diff::format_diff(
                        &format!("{:#?}", left_val),
                        &format!("{:#?}", right_val),
                    );
                    panic!("assertion failed: `left == right`\n{}", diff);
                }
            }
        }
    };
}
```

## Design Patterns

### Proc Macro Best Practices

1. **Thin Wrappers**: Keep proc macro crate minimal
2. **Use proc_macro2**: Enables testing
3. **Preserve Spans**: Better error messages
4. **Proper Error Handling**: Use `syn::Error` with spans

### Testing Strategy

1. **Unit Tests** (in `tust-core/tests/`):
   ```rust
   #[test]
   fn test_parse_simple_case() {
       let input = quote! {
           #[test_case(1, 2)]
           fn test(a: i32, b: i32) {}
       };
       let result = parse_test_case(input).unwrap();
       assert_eq!(result.cases.len(), 1);
   }
   ```

2. **Compile-Fail Tests** (using `trybuild`):
   ```
   tests/ui/
   ├── invalid_arity.rs       # Should fail
   ├── invalid_arity.stderr   # Expected error
   ├── valid_basic.rs         # Should pass
   ```

3. **Integration Tests**:
   - Test full macro expansion
   - Verify generated code executes correctly

### Error Messages

Use proper spans and provide helpful context:

```rust
// Bad
return Err(syn::Error::new(
    Span::call_site(),
    "invalid arguments"
));

// Good
return Err(syn::Error::new(
    args.span(),
    format!(
        "expected {} arguments but found {}; \
        note: function signature has {} parameters",
        expected, found, params
    )
));
```

## Code Generation Strategies

### Parameterized Tests

For each test case, generate a separate `#[test]` function:

```rust
// Input:
#[test_case(1, 2, 3)]
#[test_case(4, 5, 9)]
fn add(a: i32, b: i32, expected: i32) {
    assert_eq!(a + b, expected);
}

// Generated:
#[test]
fn add_case_0() {
    let (a, b, expected) = (1, 2, 3);
    assert_eq!(a + b, expected);
}

#[test]
fn add_case_1() {
    let (a, b, expected) = (4, 5, 9);
    assert_eq!(a + b, expected);
}
```

### Fixtures

Inject setup code at the start of tests:

```rust
// Input:
#[fixture]
fn database() -> Database {
    Database::new()
}

#[test]
fn my_test(#[fixture] database: Database) {
    // ...
}

// Generated:
#[test]
fn my_test() {
    let database = {
        Database::new()
    };
    // original test body
}
```

### Async Support

Detect and wrap with appropriate runtime:

```rust
// Input:
#[tust::test]
async fn my_test() {
    // ...
}

// Generated (if tokio detected):
#[tokio::test]
async fn my_test() {
    // ...
}
```

## Performance Considerations

1. **Compilation Speed**:
   - Keep proc macros thin
   - Minimize dependencies in macro crates
   - Use feature flags for optional functionality

2. **Runtime Performance**:
   - Lazy fixture initialization
   - Minimize overhead in assertion macros
   - Use const functions where possible

3. **Binary Size**:
   - Feature-gate optional components
   - Avoid monomorphization bloat

## Future Considerations

### Extensibility

- **Plugin System**: Allow users to define custom assertions
- **Reporter API**: Pluggable test reporters
- **Custom Matchers**: Trait-based matcher system

### Integration Points

- **Coverage Tools**: Hooks for coverage integration
- **IDE Support**: LSP extensions for better tooling
- **CI/CD**: Output formats (JUnit XML, JSON)

### Compatibility

- **Existing Tests**: Work alongside `#[test]`
- **Other Frameworks**: Composable with rstest, proptest, etc.
- **Stable Rust**: No nightly features required

## References

For implementation details, see:
- [syn documentation](https://docs.rs/syn)
- [quote documentation](https://docs.rs/quote)
- [proc-macro2 documentation](https://docs.rs/proc-macro2)
- [Rust Procedural Macros](https://doc.rust-lang.org/reference/procedural-macros.html)
