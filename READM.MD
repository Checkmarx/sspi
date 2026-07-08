# sspi (Checkmarx fork)

This repository provides Go packages for using Microsoft SSPI on Windows.

## What this repo provides

- Windows SSPI primitives in package `sspi`
- Authentication packages:
  - `github.com/checkmarx/sspi/negotiate` (SPNEGO/Negotiate)
  - `github.com/checkmarx/sspi/kerberos` (Kerberos)
  - `github.com/checkmarx/sspi/ntlm` (NTLM)
  - `github.com/checkmarx/sspi/schannel` (TLS/Schannel helpers)

## Platform support

- Windows only
- Most source files are guarded with Windows build tags

## Enabling Kerberos authentication in ast-cli

Use this library when ast-cli needs to authenticate to a Kerberos-enabled HTTP endpoint
with `Authorization: Negotiate <token>`.

### Prerequisites

1. ast-cli is running on Windows.
2. Client machine can obtain Kerberos tickets (domain-joined or valid AD credentials).
3. Service SPN is correctly registered for the target service account.
4. Hostname, DNS, and system time are correct (Kerberos is sensitive to clock skew).

### Recommended package choice

- Prefer `negotiate` for HTTP Negotiate flows.
- Use `kerberos` when you explicitly need Kerberos package behavior.

### Minimal client flow for HTTP Negotiate

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

func doKerberosRequest(rawURL string) error {
	u, err := url.Parse(rawURL)
	if err != nil {
		return err
	}

	// Standard HTTP SPN format.
	targetName := "HTTP/" + strings.ToUpper(u.Host)

	cred, err := negotiate.AcquireCurrentUserCredentials()
	if err != nil {
		return err
	}
	defer cred.Release()

	ctx, outToken, err := negotiate.NewClientContext(cred, targetName)
	if err != nil {
		return err
	}
	defer ctx.Release()

	req, err := http.NewRequest("GET", rawURL, nil)
	if err != nil {
		return err
	}
	req.Header.Set("Authorization", "Negotiate "+base64.StdEncoding.EncodeToString(outToken))

	resp, err := http.DefaultClient.Do(req)
	if err != nil {
		return err
	}
	defer resp.Body.Close()

	if resp.StatusCode == http.StatusUnauthorized {
		// In multi-leg auth, parse server token from WWW-Authenticate,
		// call ctx.Update(serverToken), then retry with the new token.
		return fmt.Errorf("received 401; continue SPNEGO token exchange")
	}

	if resp.StatusCode < 200 || resp.StatusCode > 299 {
		return fmt.Errorf("unexpected status: %s", resp.Status)
	}
	return nil
}
```

See existing negotiation tests for complete token exchange examples:

- `negotiate/http_test.go`
- `kerberos/kerberos_test.go`

## Installing

```powershell
go get github.com/checkmarx/sspi
```

## Running tests

Run on a Windows machine:

```powershell
go test ./...
```

Some tests require environment-specific inputs (for example SPN or URL) and may be skipped if not provided.

## License

BSD-style license. See `LICENSE`.
