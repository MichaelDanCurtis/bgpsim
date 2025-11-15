# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

### Added
- Initial fork from [nsg-ethz/bgpsim](https://github.com/nsg-ethz/bgpsim)
- CI workflows for lint (rustfmt + clippy), tests, and benchmarks
- Dependabot configuration for GitHub Actions
- CODEOWNERS for repository governance
- CONTRIBUTING.md with development guidelines
- CHANGELOG.md for tracking changes

### Changed
- Updated workflows to use modern GitHub Actions (dtolnay/rust-toolchain, actions/checkout@v4)

## [Upstream Syncs]

### 2025-11-15 - Initial Fork
- Forked from [nsg-ethz/bgpsim](https://github.com/nsg-ethz/bgpsim) at commit [TBD]
- Divergence: Added governance files, CI enhancements, and roadmap for plugin architecture