# Debtmap GitHub Action

[![GitHub Marketplace](https://img.shields.io/badge/Marketplace-Debtmap-blue.svg?style=flat&logo=github)](https://github.com/marketplace/actions/debtmap-code-complexity-analyzer)
[![License](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)

Analyze code complexity and identify high-risk areas in your Rust codebase. Debtmap helps you prioritize refactoring and testing efforts by identifying complex code and correlating it with test coverage to find genuinely risky functions.

## Features

- 📊 **Complexity Analysis**: Identifies overly complex code using cyclomatic and cognitive complexity metrics
- 🎯 **Risk Prioritization**: Combines complexity with test coverage to find high-risk code (high complexity + low coverage)
- 🔍 **God Object Detection**: Finds classes and modules with too many responsibilities
- 📉 **False Positive Reduction**: Uses entropy analysis to distinguish genuine complexity from repetitive patterns
- 📋 **Actionable Recommendations**: Specific guidance on what to refactor or test first
- 🚀 **Fast**: Uses pre-built Rust binaries, no compilation required
- ⚙️ **Configurable**: Use `.debtmap.toml` in your repo for custom thresholds and settings

## Quick Start

### Basic Analysis (No Coverage)

```yaml
- name: Analyze Code Complexity
  uses: iepathos/debtmap-action@v1
  with:
    path: .
```

### With Coverage (Recommended)

```yaml
# Generate coverage data first
- name: Generate Coverage
  run: cargo llvm-cov --lcov --output-path lcov.info

# Analyze with risk correlation
- name: Analyze Code Complexity
  uses: iepathos/debtmap-action@v1
  with:
    lcov-file: lcov.info
```

## Inputs

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `lcov-file` | Path to LCOV coverage file (enables risk correlation) | No | - |
| `path` | Path to the project to analyze | No | `.` |
| `debtmap-version` | Debtmap version to use (e.g., `latest`, `v0.1.0`) | No | `latest` |

## Outputs

| Output | Description |
|--------|-------------|
| `report-path` | Path to the generated markdown report |

## Configuration

For advanced configuration (thresholds, filters, etc.), create a `.debtmap.toml` file in your repository root. See the [debtmap documentation](https://iepathos.github.io/debtmap/configuration.html) for all available options.

## Usage Examples

### Complete CI Workflow

```yaml
name: Code Quality Analysis
on: [push, pull_request]

jobs:
  analyze:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Setup Rust
        uses: dtolnay/rust-toolchain@stable

      - name: Install cargo-llvm-cov
        uses: taiki-e/install-action@cargo-llvm-cov

      - name: Generate Coverage
        run: cargo llvm-cov --lcov --output-path lcov.info

      - name: Analyze Complexity
        uses: iepathos/debtmap-action@v1
        with:
          lcov-file: lcov.info

      - name: View Report
        run: cat debtmap-report.md
```

### Without Coverage

```yaml
name: Complexity Check
on: [push]

jobs:
  analyze:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Analyze Complexity
        uses: iepathos/debtmap-action@v1
```

### With Custom Configuration

```yaml
# .debtmap.toml in your repo
[thresholds]
complexity = 15
duplication = 100

[god_object]
max_methods = 25
max_fields = 10
```

```yaml
name: Custom Analysis
on: [push]

jobs:
  analyze:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Generate Coverage
        run: cargo llvm-cov --lcov --output-path lcov.info

      - name: Analyze with Custom Config
        uses: iepathos/debtmap-action@v1
        with:
          lcov-file: lcov.info
          # Config comes from .debtmap.toml
```

### Alternative Coverage Tools

Debtmap accepts any LCOV-format coverage file. Here are common ways to generate it:

```yaml
# With cargo-tarpaulin (Linux only)
- name: Generate Coverage
  run: |
    cargo install cargo-tarpaulin
    cargo tarpaulin --out Lcov --output-dir ./coverage
- uses: iepathos/debtmap-action@v1
  with:
    lcov-file: ./coverage/lcov.info

# With grcov
- name: Generate Coverage
  run: |
    cargo install grcov
    export CARGO_INCREMENTAL=0
    export RUSTFLAGS="-Cinstrument-coverage"
    cargo build
    cargo test
    grcov . -s . --binary-path ./target/debug/ -t lcov --branch --ignore-not-existing -o lcov.info
- uses: iepathos/debtmap-action@v1
  with:
    lcov-file: lcov.info

# With cargo-llvm-cov (Recommended)
- uses: taiki-e/install-action@cargo-llvm-cov
- name: Generate Coverage
  run: cargo llvm-cov --lcov --output-path lcov.info
- uses: iepathos/debtmap-action@v1
  with:
    lcov-file: lcov.info
```

## Understanding the Analysis

Debtmap identifies and prioritizes technical debt using:

- **Complexity Metrics**: Cyclomatic and cognitive complexity to find hard-to-maintain code
- **Coverage Correlation**: Combines complexity with test coverage to identify high-risk areas
- **God Object Detection**: Finds classes/modules with too many responsibilities
- **Pattern Recognition**: Reduces false positives from repetitive but simple code
- **Actionable Recommendations**: Tells you exactly what to refactor or test first

The output shows top priority items ranked by risk score, with specific guidance on what actions to take.

## Binary Releases

The action automatically downloads pre-built binaries from the [debtmap releases](https://github.com/iepathos/debtmap/releases). Supported platforms:
- Linux x86_64
- Linux aarch64
- macOS x86_64
- macOS aarch64 (Apple Silicon)

## Troubleshooting

### Coverage File Not Found

If using coverage, ensure your coverage generation step completes successfully:
```yaml
- name: Generate Coverage
  run: cargo llvm-cov --lcov --output-path lcov.info

- name: Verify Coverage File
  run: ls -la lcov.info

- name: Run Debtmap
  uses: iepathos/debtmap-action@v1
  with:
    lcov-file: lcov.info
```

### Analysis Takes Too Long

For very large projects:
- Use `--max-files` in a `.debtmap.toml` config to limit analysis
- Disable expensive features like call graph analysis if not needed
- Consider analyzing only changed files in PRs

### High Memory Usage

For large projects, ensure adequate runner resources:
```yaml
runs-on: ubuntu-latest  # 7GB RAM
# or
runs-on: ubuntu-latest-16-core  # More resources
```

## Contributing

Contributions are welcome! Please see [CONTRIBUTING.md](CONTRIBUTING.md) for details.

## License

This action is distributed under the MIT License. See [LICENSE](LICENSE) for details.

## Support

- 🐛 [Report issues](https://github.com/iepathos/debtmap-action/issues)
- 💡 [Request features](https://github.com/iepathos/debtmap-action/issues/new?labels=enhancement)
- 📖 [Read debtmap docs](https://github.com/iepathos/debtmap)