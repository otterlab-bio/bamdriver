# bamdriver

**A shared pure-Go library for BAM, BGZF, indexing, sorting, and alignment helpers.**

`bamdriver` is the low-level I/O layer extracted for reuse by `xenofilx` and `pairbam`. It is a library first; ordinary workflow users should invoke the higher-level operator that owns their data contract.

## Proof

The package tests need neither a network nor a reference genome:

```text
$ go test ./...
ok  	github.com/otterlab-bio/bamdriver/cmd/bamroundtrip	(cached)
ok  	github.com/otterlab-bio/bamdriver/cmd/nmoracle	(cached)
ok  	github.com/otterlab-bio/bamdriver/pkg/bamnative	(cached)
ok  	github.com/otterlab-bio/bamdriver/pkg/bgzip	(cached)
```

The two tools in that list are the reason this is a shared library rather than a copy per tool:
`bamroundtrip` decodes a BAM, re-encodes it, and rebuilds its BAI as a preservation check, and
`nmoracle` is the independent edit-distance oracle that `xenofilx` scores against.

## Packages

- `pkg/bgzip` — BGZF reading, writing, and virtual-offset helpers.
- `pkg/bamnative` — BAM reading/writing, coordinate sorting, BAI indexing, FASTA access, and NM-related helpers.

## Diagnostic CLIs

These are evidence/debugging tools, not workflow operators:

```bash
# Decode/re-encode a BAM, emit a JSON report, and optionally build BAI.
go run ./cmd/bamroundtrip roundtrip \
  --input input.bam --output roundtrip.bam --report roundtrip.json --index

# Compare two BAM streams and write a JSON comparison report.
go run ./cmd/bamroundtrip compare \
  --left expected.bam --right actual.bam --report compare.json

# Independently recompute conventional/bisulfite NM and Xenofilx scores.
go run ./cmd/nmoracle \
  --input input.bam --reference genome.fa --report nm-summary.json \
  --records per-record.tsv --limit 0
```

## Use from a consumer

During local multi-repository development:

```go
require github.com/otterlab-bio/bamdriver v0.0.0
replace github.com/otterlab-bio/bamdriver => ../bamdriver
```

For a published dependency, use a tagged version and remove the local `replace` directive.

## Gate A reproducibility check

The repository includes a fixture-based BAM preservation workflow at `.github/workflows/gate-a.yml`. It downloads the pinned RNA-PDX `SRR30880970` hg38 BAM/BAI fixture from `fallingstar10/otter-data`, verifies the published SHA-256 values, runs the `bamroundtrip` decode/encode path, creates a new BAI, compares the decoded header and ordered canonical record stream, and independently runs `samtools quickcheck` and `idxstats` through `enva`.

The workflow intentionally starts from an existing alignment fixture. It does not perform read mapping or alignment on GitHub-hosted runners. Round-trip reports, comparison reports, logs, generated BAM/BAI files, and tool-version evidence are uploaded as an Actions artifact, including when a preceding step fails.

## Validation

```bash
go test ./...
go vet ./...
```

When changing exported BAM or BGZF behavior, validate at least one downstream consumer such as `xenofilx` or `pairbam`.

## Repository scripts

- `scripts/bootstrap_repo.sh [remote_url]` initializes a standalone repository and optional origin.
- `scripts/release.sh vX.Y.Z` runs tidy/tests and creates a release tag.
- `scripts/update_consumer.sh /path/to/consumer vX.Y.Z` switches a consumer from a local replacement to a published tag.

## License and repository

MIT · [otterlab-bio/bamdriver](https://github.com/otterlab-bio/bamdriver)
