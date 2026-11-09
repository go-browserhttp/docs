# gitcorsproxy

A CORS reverse proxy for git smart-HTTP, so a Go/wasm git client in a browser can reach remotes that speak the protocol correctly but send no `Access-Control-Allow-Origin` (GitHub, a bare Forgejo). Standard library only, CGO=0.

It stores nothing: the browser holds the user's token and sends it on each request; the proxy relays it upstream and forgets it.

## Behaviour

- `Access-Control-Allow-Origin` echoes the request `Origin` only if it is in the configured list, never `*`.
- Only allow-listed upstream hosts are reachable; resolved IPs in private, loopback, link-local or metadata ranges are refused.
- Per-client-IP rate limit, response-size cap and per-request timeout.

## Authorization handling (v0.4.0)

- A response to a request that carried `Authorization` is `Cache-Control: private, no-store` with `Vary: Authorization`, whatever the upstream said.
- The Authorization header is never logged, query string included.
- It is never followed through a redirect: the proxy never follows one and relays no `Location`.

## Hardening (v0.5.0)

- A refused host segment (which may be `user:token@host`) is logged only as a strict, bounded `hostname[:port]` with the userinfo dropped, otherwise as `<invalid-host>`.
- `-trusted-hops` defaults to auto: `1` behind a reverse proxy, `0` when the proxy serves TLS itself (`-tls-cert` and `-tls-key`), where a client could otherwise choose its own `X-Forwarded-For` address. A non-zero value with TLS prints a warning.
- The host is resolved once; the connection goes to the IP that was checked, and private, loopback, link-local, multicast, unspecified and metadata addresses are refused at dial time (no DNS rebinding). The default transport ignores `HTTP_PROXY`/`HTTPS_PROXY`.
- Repository paths with empty, `.` or `..` segments (also percent-encoded), control characters or backslashes, and `info/refs` queries without exactly one `service` value, are refused `400`. The route parser is fuzzed.

!!! warning
    The operator of a proxy can read every token that passes through it. Using a public proxy means trusting its operator. Prefer a forge REST API, which needs no proxy (see [browsergit](browsergit.md)), or run your own proxy.

## Run

```sh
go run ./cmd/gitcorsproxy -origins https://go-tex.github.io -hosts github.com
```

`-origins` and `-hosts` are required; each flag has a `GITCORSPROXY_*` environment variable.

## Links

- Source — <https://github.com/go-browserhttp/gitcorsproxy>
- API reference — <https://pkg.go.dev/github.com/go-browserhttp/gitcorsproxy>
