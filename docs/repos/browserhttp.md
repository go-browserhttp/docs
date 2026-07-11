# browserhttp

Pure-Go `http.Client` presenting a Chrome TLS fingerprint via uTLS. CGO=0.

A drop-in `http.Client` whose TLS ClientHello matches a real Chrome build, so endpoints that gate on the TLS fingerprint let it through. It pairs the uTLS handshake with a browser User-Agent and a warmed cookie jar.

## Highlights

- **CGO_ENABLED=0**, no host web view and no headless browser.
- Real Chrome TLS fingerprint (ciphers, extensions, curves) via uTLS.
- Browser User-Agent and a warmed cookie jar out of the box.
- Identical behaviour on macOS, Linux and Windows.
- Extracted from `go-news-reader/reader`; reusable by any browser-shaped fetcher.

## Example

```go
c := browserhttp.NewClient(30 * time.Second)
resp, err := c.Get("https://www.reddit.com/r/golang/hot.json")
```

## Install

```sh
go get github.com/go-browserhttp/browserhttp
```

Requires Go 1.26 or newer.

## Links

- Source — <https://github.com/go-browserhttp/browserhttp>
- API reference — <https://pkg.go.dev/github.com/go-browserhttp/browserhttp>
