# Contributing to BGPemu2

Thank you for your interest in contributing! This project is a fork of [nsg-ethz/bgpsim](https://github.com/nsg-ethz/bgpsim) with extended scope for BGP emulation and testing.

## Quick Start

1. **Fork & Clone**
   ```bash
   gh repo fork MichaelDanCurtis/bgpsim --clone --remote
   cd bgpsim
   ```

2. **Install Rust**
   ```bash
   curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
   rustup default nightly
   rustup component add rustfmt clippy
   ```

3. **Build & Test**
   ```bash
   cargo build --all-features
   cargo test --all-features --manifest-path bgpsim/Cargo.toml
   cargo fmt --all -- --check
   cargo clippy --all-features --manifest-path bgpsim/Cargo.toml -- --deny warnings
   ```

## Development Workflow

### Branch Naming
Use the format: `[###-feature-name]`

Example: `001-add-gobgp-adapter`

### Before Submitting a PR

- [ ] Write tests first (TDD required per constitution)
- [ ] Run `cargo fmt --all`
- [ ] Run `cargo clippy --all-features --manifest-path bgpsim/Cargo.toml -- --deny warnings`
- [ ] Run `cargo test --all-features --manifest-path bgpsim/Cargo.toml`
- [ ] Update CHANGELOG.md under `[Unreleased]`
- [ ] Include constitution check notes in PR description

### Constitution Check

All PRs must verify compliance with [BGPemu2 Constitution](https://github.com/MichaelDanCurtis/BGPemu2/blob/main/.specify/memory/constitution.md) principles:

- **Test-First**: Tests written and failing before implementation
- **Library-First**: Features as standalone, testable libraries
- **CLI-First**: Expose functionality via CLI with text I/O
- **RFC Compliance**: Protocol behavior aligns with RFCs; deviations documented
- **Safety & Isolation**: Emulations run in isolated environments
- **Reproducibility**: Scenarios defined declaratively with seeds

### Commit Messages

Follow [Conventional Commits](https://www.conventionalcommits.org/):

- `feat:` new feature
- `fix:` bug fix
- `docs:` documentation only
- `test:` adding or updating tests
- `ci:` CI/CD changes
- `refactor:` code refactoring
- `perf:` performance improvements

### Pull Request Process

1. Create a feature branch from `main`
2. Make changes following TDD (tests → implementation → refactor)
3. Update documentation and CHANGELOG
4. Push branch and open PR with:
   - Clear description of changes
   - Constitution compliance notes
   - Test coverage summary
5. Address review feedback
6. Squash commits if requested

## Project Structure

```
bgpsim/
├── bgpsim/           # Core simulation library
├── bgpsim-macros/    # Procedural macros
├── bgpsim-web/       # Web interface
├── .github/          # CI workflows and repo config
└── Cargo.toml        # Workspace definition
```

## Testing Strategy

- **Unit Tests**: `cargo test` (per-module)
- **Integration Tests**: `tests/` directories
- **Contract Tests**: Verify plugin interfaces
- **Doc Tests**: Examples in doc comments

## Code Style

- Follow Rust idioms and API guidelines
- Use `rustfmt` (enforced in CI)
- Address all `clippy` warnings (CI runs with `--deny warnings`)
- Document public APIs with examples

## Upstream Relationship

This fork maintains periodic syncs with [nsg-ethz/bgpsim](https://github.com/nsg-ethz/bgpsim):

- Prefer upstream-first fixes when feasible
- Track divergence in CHANGELOG under `[Upstream Syncs]`
- Run contract and regression tests before merging upstream updates

## Getting Help

- Open an [issue](https://github.com/MichaelDanCurtis/bgpsim/issues) for bugs or feature requests
- Check the [upstream documentation](https://bgpsim.github.io/) for core concepts
- Review the [BGPemu2 Constitution](https://github.com/MichaelDanCurtis/BGPemu2/blob/main/.specify/memory/constitution.md) for project principles

## License

See [LICENSE](LICENSE) file. Contributions are accepted under the same license as the original project.