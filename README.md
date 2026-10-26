# zanadir-action

GitHub Action for [zanadir](https://github.com/MustacheCase/zanadir) — it scans a
repository's CI configuration and reports the categories of tooling it is
missing, with concrete suggestions for each.

## Usage

```yaml
- uses: actions/checkout@v4
  with:
    fetch-depth: 0

- uses: mustachecase/zanadir-action@v1.29
  with:
    dir: .
```

`fetch-depth: 0` matters — zanadir inspects the repository as a git checkout.

## Effectiveness

Presence is not protection. A scanner wired up so it can never fail a build
leaves a repository worse off than one with no scanner, because everybody
believes the control is there. The action reports those too:

```
3 covered categories are weakened:
  Linter is partial (.github/workflows/pushonly.yml)
  SCA is advisory (.github/workflows/nightly.yml)
  Secrets Detection is advisory (.github/workflows/ci.yml)
```

`continue-on-error: true`, a scanner that only runs on a schedule, a linter
that never sees a pull request, `trivy --exit-code 0`, `soft_fail: true`,
GitLab's `allow_failure` and `when: manual` — all of them used to count as
covered.

A weakened control does not fail the job unless you set `fail-on-weakened`.
`enforce` on its own still acts only on categories with no tooling at all, so
upgrading does not start failing builds that passed.

Dependabot and Renovate are read from their configuration files and graded
advisory: they open pull requests rather than failing builds.

### Required checks

A scanner that runs on every pull request still blocks nothing if its check is
not required to merge. Set `required-checks` and zanadir reads the default
branch's rulesets and branch protection, so a control enforces only when its
check must pass, or when code scanning blocks a merge on its alerts:

```yaml
- uses: mustachecase/zanadir-action@v1.29
  with:
    dir: .
    required-checks: true
```

```
Merge settings for main:
  Required to merge: Static analysis, Unit tests.
  Code scanning alerts block a merge from: zizmor.
```

It uses the job's `github.token`, read-only, so it works on private
repositories too.

## Inputs

| Input | Description | Default |
|---|---|---|
| `dir` | Path to the repository directory (**required**) | — |
| `output` | Output format: `table`, `json`, `sarif` or `markdown` | `table` |
| `output-file` | Write the report to this file instead of stdout | stdout |
| `badge` | Write a shields.io endpoint badge of the coverage score to this path | none |
| `excluded-categories` | Comma-separated categories to skip entirely | none |
| `enforce` | Fail the job when any category is uncovered | `false` |
| `fail-on` | Fail only when these specific categories are uncovered | none |
| `fail-on-weakened` | Also fail when a category has tooling that cannot fail a build | `false` |
| `baseline` | Path to a file of already-accepted gaps | none |
| `write-baseline` | Record the current gaps as accepted, then exit successfully | `false` |
| `debug` | Verbose logging | `false` |
| `config` | YAML file of settings and custom rules; an input set here wins over the file | none |
| `required-checks` | Grade controls against the checks the default branch requires to merge | `false` |
| `token` | Token for reading the merge settings with `required-checks` | `github.token` |

## Config file and custom rules

Keep settings in the repository instead of in every workflow, and teach
zanadir about in-house scanners it would otherwise report as missing:

```yaml
# .zanadir.yml
excluded-categories: [Performance Testing]
fail-on-weakened: true
rules:
  - id: acme-scan
    applyOn: [Job.Run]
    categories: [SAST]
    regex: '\bacme-scan\b'
```

```yaml
- uses: mustachecase/zanadir-action@v1.29
  with:
    dir: .
    config: .zanadir.yml
```

An unknown key, a misspelt category or a custom rule reusing a built-in `id`
fails the step. See the
[zanadir README](https://github.com/MustacheCase/zanadir#config-file) for every
key.

## Publishing to GitHub code scanning

Emit SARIF to a file and hand it to `upload-sarif`, and uncovered categories
appear in the repository's **Security** tab alongside your other scanners:

```yaml
- uses: mustachecase/zanadir-action@v1.29
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
- uses: mustachecase/zanadir-action@v1.29
  with:
    dir: .
    fail-on: SAST,Secrets Detection
```

To adopt enforcement on an existing repository, record today's gaps as accepted
and fail only on new ones:

```yaml
- uses: mustachecase/zanadir-action@v1.29
  with:
    dir: .
    enforce: true
    baseline: .zanadir-baseline.yaml
```

Generate that file once with `write-baseline: true`, or locally with
`zanadir scan --dir . --write-baseline`, and commit it.

## Categories

SCA · Secrets Detection · License Compliance · End Of Life · Coverage · Linter ·
Performance Testing · Unit Tests · SAST · IaC Security · Supply Chain ·
Workflow Security

A category with nothing to scan is left out of the score rather than suggested
— a repository with no Terraform, Helm chart, Kubernetes manifest or
Dockerfile is scored without IaC Security, and one with no GitHub Actions
without Workflow Security, since every tool in that category reads that
format. The report says which.

## Versioning

Each release pins a specific zanadir image, so the action's behaviour only
changes when you bump it. `v1.29` runs `zanadir:0.23.0`.
