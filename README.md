# zanadir-action

GitHub Action for [zanadir](https://github.com/MustacheCase/zanadir) — it scans a
repository's CI configuration and reports the categories of tooling it is
missing, with concrete suggestions for each.

## Usage

```yaml
- uses: actions/checkout@v4
  with:
    fetch-depth: 0

- uses: mustachecase/zanadir-action@v1.6
  with:
    dir: .
```

`fetch-depth: 0` matters — zanadir inspects the repository as a git checkout.

## Inputs

| Input | Description | Default |
|---|---|---|
| `dir` | Path to the repository directory (**required**) | — |
| `output` | Output format: `table`, `json` or `sarif` | `table` |
| `output-file` | Write the report to this file instead of stdout | stdout |
| `excluded-categories` | Comma-separated categories to skip entirely | none |
| `enforce` | Fail the job when any category is uncovered | `false` |
| `fail-on` | Fail only when these specific categories are uncovered | none |
| `baseline` | Path to a file of already-accepted gaps | none |
| `write-baseline` | Record the current gaps as accepted, then exit successfully | `false` |
| `debug` | Verbose logging | `false` |

## Publishing to GitHub code scanning

Emit SARIF to a file and hand it to `upload-sarif`, and uncovered categories
appear in the repository's **Security** tab alongside your other scanners:

```yaml
- uses: mustachecase/zanadir-action@v1.6
  with:
    dir: .
    output: sarif
    output-file: zanadir.sarif

- uses: github/codeql-action/upload-sarif@v3
  with:
    sarif_file: zanadir.sarif
```

Note `output-file` rather than a shell redirect: this is a Docker action with no
shell, and redirecting would also capture the debug log into the report.

## Failing the build

`enforce` fails on any gap, which is usually too blunt to adopt at once.
`fail-on` blocks only the categories you care about:

```yaml
- uses: mustachecase/zanadir-action@v1.6
  with:
    dir: .
    fail-on: SAST,Secrets Detection
```

To adopt enforcement on an existing repository, record today's gaps as accepted
and fail only on new ones:

```yaml
- uses: mustachecase/zanadir-action@v1.6
  with:
    dir: .
    enforce: true
    baseline: .zanadir-baseline.yaml
```

Generate that file once with `write-baseline: true`, or locally with
`zanadir scan --dir . --write-baseline`, and commit it.

## Categories

SCA · Secrets Detection · License Compliance · End Of Life · Coverage · Linter ·
Performance Testing · Unit Tests · SAST · IaC Security

## Versioning

Each release pins a specific zanadir image, so the action's behaviour only
changes when you bump it. `v1.6` runs `zanadir:0.2.0`.
