# bamdriver Release Checklist

## 1) Pre-release validation

Run in `bamdriver`:

```bash
go mod tidy
go test ./...
```

Run consumer smoke tests:

```bash
cd ../xenofilx && go test ./...
cd ../pairbam && go test ./...
```

## 2) Tag and publish

In the `bamdriver` repo:

```bash
VERSION=vX.Y.Z
./scripts/release.sh "$VERSION"
git push origin main --tags
```

## 3) Upgrade consumers

In `xenofilx/go.mod` and `pairbam/go.mod`:

1. Change the existing bamdriver requirement (currently `v1.0.0` in both
   consumers) to the release being published:

```go
require github.com/otterlab-bio/bamdriver vX.Y.Z
```

2. Remove local replace:

```go
replace github.com/otterlab-bio/bamdriver => ../bamdriver
```

3. Run:

```bash
go mod tidy
go test ./...
```

Or from `bamdriver`:

```bash
VERSION=vX.Y.Z
./scripts/update_consumer.sh ../xenofilx "$VERSION"
./scripts/update_consumer.sh ../pairbam "$VERSION"
```

## 4) CI gate (recommended)

Add checks in each consumer pipeline:

```bash
go test ./...
# if available
samtools quickcheck <output.bam>
samtools view -H <output.bam> >/dev/null
samtools view -c <output.bam> >/dev/null
```

## 5) Post-release verification

- Confirm no compatibility shims or legacy import paths remain.
- Keep package paths rooted at `github.com/otterlab-bio/bamdriver`.
