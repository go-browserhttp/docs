# browsergit

The git layer shared by the go-tex, go-slides and notebook playgrounds: load a project from GitHub, GitLab or Gitea/Forgejo and propose changes back as a new branch plus a pull request. Pure Go, runs in the browser (js/wasm).

Released as v0.4.0. CGO=0, BSD-3-Clause.

## Packages

| Package | What it is |
|---|---|
| `remote` | The `Remote` interface (`Load`, `Propose`, `Refresh`), `Base`, path/branch/repository validation, size limits, and `Git`, a `Remote` over git smart HTTP. |
| `forge` | `Remote` over the REST APIs of GitHub, GitLab and Gitea/Forgejo. |
| `secret` | `Token`: a credential that prints as `[redacted]` and refuses every encoder. |
| `browsergit` (root) | go-git v5 in memory, smart HTTP over the browser's Fetch: clone, status, commit, pull, push, snapshot. |
| `gitd` | `Remote` over [slides-gitd](https://github.com/go-slides/gitd), the self-hosted backend a playground is served behind. The page holds no forge credential (since v0.4.0). |
| `gitrpc` | The JSON protocol between a page and a git web worker. |

## Example

```go
r, _ := forge.New(forge.Config{Kind: forge.GitHub, API: "https://api.github.com", Token: secret.New(tok)})
files, base, _ := r.Load(ctx, "owner/repo", "", "slides")
files["deck.md"] = edited
res, _ := r.Propose(ctx, base, files, remote.Proposal{
    Branch: remote.BranchName("slides/", "Fix intro", time.Now()),
    Title:  "Fix intro",
})
// res.URL is the pull request
```

## Reaching a forge

| Forge | Use | Token |
|---|---|---|
| GitHub | `forge.GitHub` (REST) | Fine-grained PAT: Metadata read, Contents read/write, Pull requests write. |
| GitLab | `forge.GitLab` (REST) | OAuth (PKCE) or PAT with `api` scope. |
| Gitea / Forgejo | `forge.Gitea` (REST) when the instance allows CORS; otherwise `remote.Git` through a CORS proxy | Access token with `write:repository`. |

`remote.Git` cannot open a pull request: it pushes the new branch and returns the forge's compare page.

## Behind a server: `gitd`

When a playground is served behind slides-gitd, the page needs no forge token at all. The `gitd` package is a `remote.Remote` (`Load`, `Propose`, `Refresh`) that talks to the page's own origin only (Fetch mode same-origin, via the new `forge.Request.SameOrigin`). The server's HttpOnly session cookie is never readable by the page, and the per-session CSRF token is kept in memory as a `secret.Token`.

```go
c, _ := gitd.New(gitd.Config{}) // see gitd.Config for the options
info, err := c.Session(ctx)     // discovery and CSRF handshake
_ = info
switch {
case errors.Is(err, gitd.ErrNoServer):
    // no server here: behave as without one (token-based forge path)
case errors.Is(err, gitd.ErrSignIn):
    // HTTP 401 (wraps remote.ErrAuth): sign in again, never fall back to a token
}
r := c.Remote("github") // by forge name: Load / Propose / Refresh through the server
```

- `Session(ctx)` is the handshake. `ErrNoServer` means behave as without a server; `ErrSignIn` means sign in again and never fall back to a token.
- The server names the proposal's branch; `Proposal.Branch` is ignored.
- `StoreCredential` and `ForgetCredential` manage the credential of user-token forges.
- Server answers are validated as hostile input, and error text is the status plus the server's error class only.
- `FuzzParseSession` and `FuzzDecodeLoad` fuzz the parsers.

## Security

- The token is never printed, serialised, redirected or echoed.
- Redirects are refused, browser requests carry no cookies, and API and web bases must be https.
- Paths and links from a server are validated; blob content must hash to its git id; sizes are bounded.
- A proposal always creates a new branch; it never writes an existing, loaded or default branch.

!!! warning
    A CORS proxy sees the Authorization header of every request through it, so its operator can read tokens. Prefer the REST paths (no proxy) or a proxy you run yourself. See [gitcorsproxy](gitcorsproxy.md).

## Install

```sh
go get github.com/go-browserhttp/browsergit
```

## Links

- Source — <https://github.com/go-browserhttp/browsergit>
- API reference — <https://pkg.go.dev/github.com/go-browserhttp/browsergit>
