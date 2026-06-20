# etamong-httperr (Rust)

> **About** — One of several shared libraries behind a personal homelab "fleet" of small apps (error handling · audit logging · encryption-at-rest · i18n · UI · …). Published to show the **design decisions** behind these cross-cutting concerns. It is authored and maintained with [Claude Code](https://www.anthropic.com/claude-code) (Anthropic's agentic CLI), not hand-written.
>
> **This is a public repository** — keep internal infrastructure details (hostnames, secret/Vault paths, private URLs, internal issue/MR references) out of code, comments, README, and commit messages.

Rust port of the etamong-playground cross-app error contract.
Sibling of [`httperr`](https://github.com/etamong-playground/httperr) (Go).

## Stack rationale

Rust because this crate's consumers are Rust apps under a fleet-language-policy
SLO-bound exception (currently an internal triage service).
The wire format mirrors the Go contract field-for-field so both produce the
same structured log line, joinable by the 8-hex `ref` across languages.

## Use

```rust
use etamong_httperr::{emit_failed, fail_body, new_ref};

// On failure: log + return body
let ref_ = emit_failed("myapp", "POST", "/triage", 500, "ops@e.c", "db: timeout");
let body = fail_body("internal error", &ref_);
// → write `body` to client; `ref_` is the join key in structured logs.
```

## Wire format

```
{"level":"error","msg":"request failed","app":"myapp","ref":"a1b2c3d4",
 "method":"POST","path":"/triage","status":500,"user":"ops@e.c","err":"db: timeout"}
```

`level` is `"error"` (≥500), `"warn"` (400-499), `"info"` otherwise — lowercased
so a single log query `| json | level="error"` matches every fleet app.

`path` should be a low-cardinality route template
(`/api/v1/sites/{slug}`, not the raw URL) to bound log cardinality.

## Related

- [`httperr`](https://github.com/etamong-playground/httperr) — Go canonical impl
- [`audit-rs`](https://github.com/etamong-playground/audit-rs) — Rust audit companion
