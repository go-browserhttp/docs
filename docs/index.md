# go-browserhttp

A pure-Go http.Client with a Chrome TLS fingerprint (uTLS) — bypass anti-bot 403s, no host web view.

`go-browserhttp` is a pure-Go, `CGO_ENABLED=0` `http.Client` that presents a real Chrome TLS fingerprint via [uTLS](https://github.com/refraction-networking/utls). Many sites 403 non-browser clients based largely on the TLS ClientHello; mimicking Chrome's ciphers, extensions and curves — plus a browser User-Agent and a warmed cookie jar — lets a plain Go client reach public endpoints with no host web view. Identical on macOS, Linux and Windows.

## Repositories

<div class="repo-grid" markdown>
<a class="repo-card" href="repos/browserhttp.md"><code>browserhttp</code><br><small>Pure-Go `http.Client` presenting a Chrome TLS fingerprint via uTLS. CGO=0.</small></a>
<a class="repo-card" href="repos/browsergit.md"><code>browsergit</code><br><small>The git layer shared by the go-tex, go-slides and notebook playgrounds: load from a forge, propose a branch plus pull request. Runs in the browser.</small></a>
<a class="repo-card" href="repos/gitcorsproxy.md"><code>gitcorsproxy</code><br><small>A CORS reverse proxy for git smart-HTTP, for Go/wasm git clients in a browser.</small></a>
</div>
