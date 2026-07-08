# Usage Guide

This guide explains how to use `github.com/checkmarx/sspi` in Windows applications.

## Overview

`sspi` exposes low-level and package-specific APIs for Windows Security Support Provider Interface:

- `github.com/checkmarx/sspi` (core primitives)
- `github.com/checkmarx/sspi/negotiate` (SPNEGO/Negotiate)
- `github.com/checkmarx/sspi/kerberos` (Kerberos)
- `github.com/checkmarx/sspi/ntlm` (NTLM)
- `github.com/checkmarx/sspi/schannel` (Schannel helpers)

## Requirements

- Windows
- Go 1.13+
- Valid local/domain credentials for the chosen authentication flow

## Install

```powershell
go get github.com/checkmarx/sspi
```

## Typical Authentication Flow

Most packages follow the same high-level sequence:

1. Acquire client credentials.
2. Create a client security context.
3. Send output token to peer.
4. Receive peer token and call `Update` until authentication is complete.
5. Release credentials and contexts.

## Negotiate Example (HTTP target)

```go
package main

import (
    "encoding/base64"
    "fmt"
    "net/http"
    "net/url"
    "strings"

    "github.com/checkmarx/sspi/negotiate"
)

func requestWithNegotiate(rawURL string) error {
    u, err := url.Parse(rawURL)
    if err != nil {
        return err
    }

    targetName := "HTTP/" + strings.ToUpper(u.Host)

    cred, err := negotiate.AcquireCurrentUserCredentials()
    if err != nil {
        return err
    }
    defer cred.Release()

    ctx, token, err := negotiate.NewClientContext(cred, targetName)
    if err != nil {
        return err
    }
    defer ctx.Release()

    req, err := http.NewRequest("GET", rawURL, nil)
    if err != nil {
        return err
    }
    req.Header.Set("Authorization", "Negotiate "+base64.StdEncoding.EncodeToString(token))

    resp, err := http.DefaultClient.Do(req)
    if err != nil {
        return err
    }
    defer resp.Body.Close()

    if resp.StatusCode == http.StatusUnauthorized {
        return fmt.Errorf("continue token exchange using WWW-Authenticate challenge")
    }

    if resp.StatusCode < 200 || resp.StatusCode > 299 {
        return fmt.Errorf("unexpected status: %s", resp.Status)
    }

    return nil
}
```

## Kerberos-Specific Notes

- Use Service Principal Names (SPN), for example `HOST/SERVERNAME` or `HTTP/SERVERNAME`.
- Ensure DNS and machine clocks are correct; Kerberos is sensitive to skew.
- For explicit credentials, use `AcquireUserCredentials(domain, username, password)`.

## NTLM Notes

- NTLM helpers provide two/three-leg token exchange APIs similar to Kerberos/Negotiate.
- For HTTP, the authorization header is `NTLM <base64-token>`.

## Message Security APIs

After a context is established, packages expose helpers for:

- `MakeSignature` / `VerifySignature`
- `EncryptMessage` / `DecryptMessage`

These are available on context types where the underlying package supports them.

## Examples and Tests in This Repo

- Kerberos context and encryption tests: [kerberos/kerberos_test.go](../kerberos/kerberos_test.go)
- Negotiate package tests: [negotiate/negotiate_test.go](../negotiate/negotiate_test.go)
- Negotiate HTTP flow: [negotiate/http_test.go](../negotiate/http_test.go)
- NTLM package tests: [ntlm/ntlm_test.go](../ntlm/ntlm_test.go)
- NTLM HTTP flow: [ntlm/http_test.go](../ntlm/http_test.go)
