# zanadir-action
GitHub action of zanadir

# 🚀 Zanadir GitHub Action

⚙️ Scan your GitHub repository using the [Zanadir](https://github.com/mustachecase/zanadir) CLI tool to identify issues related to get CI/CD Recommendations

This action wraps the `zanadir` CLI and can be used as part of your CI workflow.

---

## 📦 Usage

```yaml
name: Run Zanadir Scan

on:
  push:
    branches: [ main ]
  pull_request:

jobs:
  zanadir:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3

      - name: Scan repository with Zanadir
        uses: mustachecase/zanadir-action@v1
        with:
          dir: .
          debug: true
          enforce: false
          output: table
```

## 🔧 Inputs

```text
Name                 Description                                                  Required  Default
----                 -----------                                                  --------  -------
dir                  Path to the GitHub repository directory to scan              ✅ Yes    -
excluded-categories  Comma-separated list of categories to exclude (e.g. sca,...) ❌ No     -
enforce              Fails the CI process if any issue is found                   ❌ No     false
debug                Run the scanner in debug mode                                ❌ No     false
output               Output format. Options: table, json                          ❌ No     table
```

## ❌ Enfore Mode

If enforce: true is set, the GitHub Action will fail the build if any issue is detected in the repository scan. This is useful for enforcing secure coding standards.

## 🐛 Debugging

Set debug: true to get verbose logs from the scanner, which can help diagnose issues in your workflow or repository setup.
