# mochi-go

Bidirectional Go module bridge for [Mochi](https://github.com/mochilang/mochi). Implements [MEP-74](https://github.com/mochilang/mochi/blob/main/website/docs/mep/mep-0074.md) (Mochi+Go package manager).

## Status

All 18 phases (0-17) LANDED at the baseline level as of 2026-05-30. Remaining work is per-phase sub-phase integration (N.1+ deferred slots: wrapper-synth wiring, MEP-54 driver wiring, mochi.lock CLI surface, live-target gates). See [MEP-74 implementation tracking](https://github.com/mochilang/mochi/blob/main/website/docs/implementation/0074/index.md) for the per-phase ledger.

## What this is

The MEP-74 bridge sits between MEP-54 (Mochi-to-Go transpiler) and MEP-57 (Mochi source-level package system). Two directions:

- **Consume**: `import go "<module>@<semver>" as <alias>` in Mochi source. The bridge ingests `go/packages.Load` via the `cmd/go-ingest` helper, lowers via a closed type table, synthesises a cgo wrapper package with `//export` directives, builds it as a `c-archive`, and exposes Go items as Mochi `extern fn` declarations.
- **Publish**: `mochi pkg publish --to=git-tag`. The bridge lowers the Mochi package via a new `TargetGoLibrary` emit (canonical-import-path-respecting Go package with `go.mod`), commits the generated tree to the user's git remote, tags with `v<semver>` so `proxy.golang.org` picks it up asynchronously, and optionally signs the sibling `v<semver>.sig` tag with Sigstore keyless cosign.

## Layout

```
apisurface/     # ApiSurface JSON parser (phase 4)
build/          # workspace + go build orchestration (phase 9)
cmd/
  go-ingest/    # go/packages ApiSurface emitter (phase 3)
cosign/         # cosign-on-sibling-tag signer (phase 13)
emit/           # Mochi extern-fn emitter (phase 7)
errors/         # bridge-side SkipReport / error envelope
goroutine/      # cgo handle pool + bridge runtime (phase 14)
library/        # TargetGoLibrary emit (phase 11)
lockfile/       # `[[go-package]]` schema + drift check (phase 10)
moduleproxy/    # proxy.golang.org client (phase 1)
monomorphise/   # `[go.monomorphise]` parser + renderer (phase 15)
publish/        # git-tag publish (phase 12)
semver/         # semantic version comparison helpers
sumdb/          # sum.golang.org transparency client (phase 2)
tinygo/         # TinyGo embedded subset (phase 16)
typemap/        # closed type-mapping table + SkipReport (phase 5)
vanity/         # vanity-import redirect resolver + wasm publish gate (phase 17)
wrapper/        # cgo wrapper synthesiser (phase 6)
```

## Building

```
go build ./...
go vet ./...
go test ./...
```

Requires Go 1.26 or later.

## License

MIT. See [LICENSE](LICENSE).
