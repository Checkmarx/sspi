# Contributing to sspi

Thank you for contributing to `github.com/checkmarx/sspi`.

This repository contains Go wrappers for Windows SSPI packages (Negotiate, Kerberos, NTLM, and Schannel).

## Code of Conduct

By participating in this project, you agree to follow our Code of Conduct in [docs/CODE_OF_CONDUCT.md](docs/CODE_OF_CONDUCT.md).

## Getting Started

1. Fork the repository.
2. Create a feature branch from `master`.
3. Make focused changes with tests/docs updates when relevant.
4. Open a pull request with a clear description of motivation and behavior changes.

## Prerequisites

- Windows (this project targets Windows SSPI APIs)
- Go 1.13 or later (see [go.mod](go.mod))

## Local Development

```powershell
go build ./...
go test ./...
```

Some tests are environment-dependent and will skip unless flags are provided.

## Running Environment-Dependent Tests

Several test packages support flags for domain credentials, SPN, or integration URL.

Examples:

```powershell
# Kerberos package tests with explicit SPN
go test ./kerberos -args -spn "HOST/MYHOST"

# Negotiate/NTLM credential tests
go test ./negotiate -args -domain MYDOMAIN -username myuser -password mypass
go test ./ntlm -args -domain MYDOMAIN -username myuser -password mypass

# HTTP integration tests for negotiate/ntlm
go test ./negotiate -run HTTP -args -url "http://server.example.com/protected"
go test ./ntlm -run HTTP -args -url "http://server.example.com/protected"
```

## Pull Request Guidelines

- Keep PRs focused on one logical change.
- Include tests for behavior changes when possible.
- Update documentation when public behavior or examples change.
- Avoid unrelated formatting-only edits.

## Reporting Issues

When filing an issue, include:

- Windows version
- Go version (`go version`)
- Package used (`negotiate`, `kerberos`, `ntlm`, or `schannel`)
- Minimal reproduction steps
- Expected behavior vs actual behavior
