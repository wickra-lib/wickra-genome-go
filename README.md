# Wickra Genome — Go

[![CI](https://github.com/wickra-lib/wickra-genome/actions/workflows/ci.yml/badge.svg)](https://github.com/wickra-lib/wickra-genome/actions/workflows/ci.yml)
[![codecov](https://codecov.io/gh/wickra-lib/wickra-genome/branch/main/graph/badge.svg)](https://codecov.io/gh/wickra-lib/wickra-genome)
[![Go module](https://raw.githubusercontent.com/wickra-lib/.github/main/profile/badges/wickra-genome/go.svg)](https://pkg.go.dev/github.com/wickra-lib/wickra-genome-go)
[![License: MIT OR Apache-2.0](https://img.shields.io/badge/license-MIT_OR_Apache--2.0-blue)](https://github.com/wickra-lib/wickra-genome#license)

**Deterministic cross-sectional market-DNA analysis for Go, over the Wickra C ABI hub via cgo.**

Wickra Genome builds a normalized feature vector per symbol and answers similarity,
clustering and anomaly queries across the cross-section — folded once in a Rust
core and byte-identical across every language. This package is the Go binding; it
consumes the C ABI hub through cgo and drives the engine over the same JSON
protocol as every other binding.

## Install

Use the published **`wickra-genome-go`** module, which bundles the prebuilt
C ABI library for every platform, so `go get` + `go build` works with no extra
steps (a C compiler is still required, as the binding uses cgo):

```bash
go get github.com/wickra-lib/wickra-genome-go
```

```go
import wickra "github.com/wickra-lib/wickra-genome-go"
```

`wickra-genome-go` is generated from the [`bindings/go`](https://github.com/wickra-lib/wickra-genome/tree/main/bindings/go)
directory by the release pipeline: it mirrors the Go sources, the vendored C ABI
header (`include/wickra_genome.h`) and the prebuilt libraries under
`lib/<goos>_<goarch>/`. On Linux/macOS the library path is baked in via rpath; on
Windows the DLL must be discoverable at run time (next to the executable or on
`PATH`).

### Building from this repository (contributors)

The `bindings/go` directory in the [wickra-genome](https://github.com/wickra-lib/wickra-genome)
repository is the development source. To build it directly, compile the C ABI and
stage the library into the per-platform directory cgo links against:

```bash
cargo build -p wickra-genome-c --release
mkdir -p lib/linux_amd64                              # match your GOOS_GOARCH
cp target/release/libwickra_genome.so    lib/linux_amd64/    # Linux
cp target/release/libwickra_genome.dylib lib/darwin_arm64/   # macOS (arm64)
cp target/release/wickra_genome.dll      lib/windows_amd64/  # Windows
```

## Quick start

```go
package main

import (
	"fmt"

	wickra "github.com/wickra-lib/wickra-genome-go"
)

func main() {
	spec := `{"features":[{"kind":"price","field":"close"}],` +
		`"symbols":["AAA","BBB","CCC"],"normalize":"z_score","metric":"euclid","seed":24333}`
	g, err := wickra.New(spec)
	if err != nil {
		panic(err)
	}
	defer g.Close()

	data := `{"AAA":[{"time":0,"open":1,"high":1,"low":1,"close":1,"volume":0}],` +
		`"BBB":[{"time":0,"open":2,"high":2,"low":2,"close":2,"volume":0}],` +
		`"CCC":[{"time":0,"open":100,"high":100,"low":100,"close":100,"volume":0}]}`
	g.Command(`{"cmd":"build","data":` + data + `}`)

	out, _ := g.Command(`{"cmd":"similar","symbol":"AAA","k":2}`)
	fmt.Println(out)
}
```

The engine lives only in the Rust core, so the same commands yield the
byte-identical similarity, clustering and anomaly results here and in every other
binding. Every handle owns native memory freed by `Close()`; a finalizer is wired
as a backstop, but call `Close()` (e.g. with `defer`) to release it promptly.

## Documentation

The full guides, quickstarts, and API reference live in the main repository and
documentation site:

- **Repository:** <https://github.com/wickra-lib/wickra-genome>
- **Docs:** <https://wickra.org>
- **Runnable examples:** [`examples/go/`](https://github.com/wickra-lib/wickra-genome/tree/main/examples/go)

Wickra ships native bindings for Python, Node.js, WASM and Rust, plus a
C ABI hub that any C-capable language (C, C++, C#, Go, Java, R) links against —
all exposing the same core from the shared, `unsafe`-forbidden Rust core.

## Security

Found a security issue? **Please don't open a public issue.** Report it privately
via the affected repository's *Security* tab (*"Report a vulnerability"*) or email
**support@wickra.org** with a subject line starting `[wickra security]`. Full
policy: <https://github.com/wickra-lib/wickra-genome/blob/main/SECURITY.md>.

## Disclaimer

Wickra Genome is analytics software, not a trading system. The values it computes
are deterministic transforms of the input data — they are not financial advice and
do not predict the market. Any use in a live trading context is at your own risk.
The library is provided **as is**, without warranty of any kind.

## License

Licensed under either of [Apache-2.0](https://github.com/wickra-lib/wickra-genome/blob/main/LICENSE-APACHE)
or [MIT](https://github.com/wickra-lib/wickra-genome/blob/main/LICENSE-MIT) at your option.
