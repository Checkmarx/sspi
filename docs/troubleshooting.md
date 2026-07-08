# Troubleshooting

Common issues when using `github.com/checkmarx/sspi`.

## Build Errors on non-Windows

### Symptom

Build fails on Linux/macOS with missing Windows syscall types.

### Cause

This repository targets Windows SSPI APIs and most files are behind Windows build tags.

### Fix

Build and run on Windows.

## "Cannot find package info" or package init failures

### Symptom

Calls such as `QueryPackageInfo` fail at runtime.

### Cause

Required SSP package is unavailable or Windows security components are not ready in the environment.

### Fix

- Run on a supported Windows environment.
- Verify package names (`NTLM`, `Negotiate`, `Kerberos`, `Schannel`) are used correctly.
- Retry in a normal user session (not stripped-down minimal environments).

## Kerberos Authentication Fails (`SEC_E_LOGON_DENIED`)

### Symptom

Kerberos or Negotiate handshake fails with logon denied.

### Common Causes

- Wrong SPN
- Server account missing SPN registration
- DNS mismatch (hostname used in SPN does not map correctly)
- Clock skew between client and KDC/server

### Fix

- Use the correct SPN format for your service, for example `HTTP/SERVER`.
- Confirm SPN registration in Active Directory.
- Ensure forward/reverse DNS resolution is correct.
- Synchronize clocks (NTP/domain time).

## Token Exchange Does Not Complete

### Symptom

`Update` loops or returns empty token before completion.

### Cause

Peer token flow is not handled correctly or wrong package/context is used.

### Fix

- Ensure every peer token is passed to the opposite side `Update` call.
- Stop only when the API reports authentication complete.
- Verify client/server are both using compatible package flows.

## HTTP Negotiate/NTLM returns repeated 401

### Symptom

Server keeps returning `401 Unauthorized`.

### Cause

- Missing/invalid `Authorization` header token
- Incomplete challenge-response loop
- Wrong target SPN for Kerberos via Negotiate

### Fix

- Base64-encode the outgoing token and set the correct scheme:
  - `Authorization: Negotiate <token>`
  - `Authorization: NTLM <token>`
- Parse challenge from `WWW-Authenticate` and continue the token exchange.
- For Kerberos over HTTP, use `HTTP/<HOST>` target name.

## User Impersonation Errors

### Symptom

`ImpersonateUser` or `RevertToSelf` fails.

### Cause

Impersonation APIs are thread-affine and require careful context lifecycle handling.

### Fix

- Lock the OS thread while impersonating (`runtime.LockOSThread`).
- Always call `RevertToSelf` before releasing the context.
- Ensure server credentials/context are valid and fully authenticated.

## Test Failures Due to Missing Flags

### Symptom

Tests are skipped or fail when domain/SPN/URL values are not provided.

### Fix

Run tests with required flags, for example:

```powershell
go test ./kerberos -args -spn "HOST/MYHOST"
go test ./negotiate -args -domain MYDOMAIN -username myuser -password mypass
go test ./ntlm -args -domain MYDOMAIN -username myuser -password mypass
go test ./negotiate -run HTTP -args -url "http://server.example.com/protected"
```

## Still Stuck?

Include the following details when opening an issue:

- Windows version
- Go version
- Exact package used (`kerberos`, `negotiate`, `ntlm`, `schannel`, or core `sspi`)
- Minimal reproducible code
- Full error value (including Windows error code)
