# etamong-httperr (Rust)

Rust port of the etamong-lab cross-app error contract (planning#188).
Sibling of [`shared/libs/httperr`](https://gitlab.com/etamong-lab/shared/libs/httperr) (Go).

## Stack rationale

Rust because this crate's consumers are Rust apps under the
[fleet-language-policy](https://gitlab.com/etamong-lab/planning/-/blob/main/wiki/concepts/fleet-language-policy.md)
SLO-bound exception (currently the `alert-ops`→`Kadabra` triage brain, planning#250).
The wire format mirrors the Go contract field-for-field so both produce the
same line on Loki and the "etamong-lab Errors" Grafana dashboard joins by
the 8-hex `ref` across languages.

## Use

```rust
use etamong_httperr::{emit_failed, fail_body, new_ref};

// On failure: log + return body
let ref_ = emit_failed("kadabra", "POST", "/triage", 500, "ops@e.c", "db: timeout");
let body = fail_body("internal error", &ref_);
// → write `body` to client; `ref_` is the join key in Grafana.
```

## Wire format

```
{"level":"error","msg":"request failed","app":"kadabra","ref":"a1b2c3d4",
 "method":"POST","path":"/triage","status":500,"user":"ops@e.c","err":"db: timeout"}
```

`level` is `"error"` (≥500), `"warn"` (400-499), `"info"` otherwise — lowercased
so a single Loki query `| json | level="error"` matches every fleet app.

`path` should be a low-cardinality route template
(`/api/v1/sites/{slug}`, not the raw URL) to bound Loki cardinality.

## Related

- planning#250 — alert-ops split (this crate is Phase 1a)
- planning#188 — cross-app error view (the contract)
- `wiki/concepts/fleet-language-policy.md`
- `shared/libs/httperr` — Go canonical impl
- `shared/libs/audit-rs` — Rust audit (planning#193) companion
