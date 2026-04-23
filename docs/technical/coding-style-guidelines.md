---
title: "Development & Coding style guidelines"
weight: 25
main:
  parent: technical
---

This document defines the coding standards, tooling expectations, and 
development practices for this project. The goal is to maintain consistency, 
readability, and long-term maintainability while keeping the rules lightweight 
and tool-driven.

## Philosophy

This project follows standard Go conventions. We prioritize the following 
principles:

- Clarity over cleverness
- Simple and readable code and designs over abstract ones in binary formats
- Tool-enforced rules over manual enforcement
- Consistency across the codebase over personal style preferences

Where possible, formatting and correctness should be checked automatically via 
tooling and editor integration.

## Code format and typing correctness

All code should be formatted and validated before being included into the main 
repository.

### Tools

- `go fmt ./...` or `make fmt` (formatting)
- `go vet ./...` or `make vet` (static correctness checks)

### Recommended workflow

Run before committing (or before merging if you find there are still issues):

```bash
make fmt       # runs go fmt ./...
make vet       # runs go vet ./...
make test      # runs unit tests
./dev/setup.sh # sets up a new cluster as indicated in local development guide
make run       # runs the application for manual testing
```

Alternatively, you can install a pre-commit hook to run these checks 
automatically before each commit. Note: This framework requires Python and pip 
to be installed, the example below assumes a globally installed pip.

```bash
pip install --user --break-system-packages pre-commit # Install pre-commit
pre-commit install                                    # Install pre-commit hooks
git commit                                            # Run checks upon commit
```

#### Integration

Several editors support automatic formatting on save. We recommend configuring 
your editor to run `go fmt` on save to ensure consistent formatting. Have a look 
at the following:

- [VSCode Go extension](https://code.visualstudio.com/docs/languages/go)
- Neovim: Install [nvim-lspconfig](https://github.com/neovim/nvim-lspconfig) and 
  then install `gopls`: `go install golang.org/x/tools/gopls@latest`

#### Rules

- All code should pass `fmt`, `vet`, and lint checks
- Avoid long discussions on formatting style or structures; upon disagreement, 
  use formatting tools for a consistent style and code correctness
- CI should enforce these checks

## Code structure and readability

Go favors explicit and simple code structure.

### Functions

- Prefer small, focused functions
- Aim for functions that do **one thing well**
- As a guideline (not a strict rule): about 20 to 50 lines per function or 
  related units (if you have many small "getter" functions, they should be 
  grouped together in a larger block); if it fits on one screen without 
  scrolling and is still easy to follow, then it is probably fine
- Avoid deep nesting; prefer early returns

#### Preferred pattern

Use early returns:

```go
if err != nil {
    return err # Or more specific information/return values
}
```

instead of deeply nested logic.

#### Function complexity

- Avoid high cyclomatic complexity: Reduce the number of (nested) `if` branches 
  inside one function. Similarly, `for` loops should not become too complicated 
  with many different conditions upon which the loop may be exited early. There 
  is no strict threshold currently pending choices regarding code style tooling 
  for this, but try not to scatter these around too much.
- Prefer splitting logic into helper functions over large monolithic blocks
- Complexity should be primarily enforced via linting tools

## Naming conventions

- Use clear, descriptive names for variables, functions and types. We may 
  occasionally review and consider renaming some of the existing resources, such 
  as how we name our controllers and the resources they manage. This is to make 
  it clearer what their scope is and how they relate to the objects/concepts 
  that they represent. At some point, a stable API should have clear naming that 
  remains the same for a longer time.
- Avoid abbreviations unless widely understood (e.g., `cfg` for configuration 
  and `err` for error are okay, but use `resourceID` instead of `rid` or 
  `configPath` instead of `cfgp`).

Package names should be:

- Lowercase with no underscores or mixed caps, while filenames can have 
  underscores but `*_test.go` is reserved for test files.
- A single word where possible, or a combination of words that are closely 
  related (a noun phrase).
- Short but meaningful.

## Error handling and reporting

Errors should be explicitly handled. Do not ignore errors using `_` unless 
justified in the code or comments. Prefer returning errors up to the caller, or 
log the error using a standard structured logging approach. If the error is 
a critical failure, consider using a panic or postmortem package.

## Documentation

Our goal with documentation is to explain why we choose do something, not just 
what we are doing. We have public-facing documentation in Markdown to explain 
milestones and technical design choices. In code, keep comments concise and 
relevant, and focus on explaining the intent and rationale behind the code 
rather than describing what the code is doing; the code itself should be clear 
enough unless it becomes very complicated. Basically, avoid redundant comments 
and documentation that restates code since this is also tough to keep up to date 
when changes are made.

- Documentation should be kept up to date with code changes.
- Public-facing features should include explanation of how to use the feature, 
  how to run/test it, and minimal example files to achieve this (such as sample 
  resources). This depends on the type of feature, but is generally recommended.
- Complex logic should include inline clarification: if you need to think about 
  it to understand it, it should be explained in the code.

### Public APIs

All exported functions, types, and packages should include GoDoc-style comments.

Example:

```go
// WriteSNMPConfig writes a new configuration file in the volume location.
func WriteSNMPConfig(fileName string) error {
```

For current development, the focus here is on the `api` directory, but we aim to 
also do this for other packages.

## Testing

Testing is necessary for all additions and changes of meaningful logic.

### Current workflow

At minimum, one of the following is expected:

- Unit tests for separate functions in the internal logic, OR
- A documented, reproducible execution flow (test by example/steps to run)

### Test expectations

- Use Go's built-in testing framework
- Tests should be deterministic and independent
- Prefer table-driven tests where appropriate

### Future direction

The project plans to evolve toward:

- Higher unit test coverage
- Integration tests
- End-to-end testing in CI pipelines

### Code review expectations

All changes must go through pull request review.

Reviewers should focus on:

- Correctness
- Readability
- Maintainability
- Test coverage
- Adherence to this document

Style debates should be resolved by existing tooling reporting, this document, 
or Go idioms, rather than with subjective discussions.

### CI pipeline expectations for pull requests

Pull requests must pass most of the pipeline jobs before merging. At minimum, 
the test and linting jobs must pass, and the documentation jobs should not fail 
(this includes checking for broken links). However, some jobs might occasionally 
fail due to external factors such as detected vulnerabilities in dependencies or 
temporary issues with external services.

In the end, it is up to the contributor and maintainer to weigh the current 
risks and benefits of merging a pull request if some steps are failing. 
Sometimes it may be better to create a separate issue to address the problems 
that are encountered, as it would otherwise affect other work as well.

## Future changes of the guidelines

This project is in an incubation and initial startup phase. These rules will 
evolve over time once the project becomes more adopted in open source.

Future versions may include:

- More clarified thresholds in code style rules
- Complexity limits via CI integration
- Expanded unit/integration testing requirements
