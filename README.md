## <img src="https://cdn.brandfetch.io/id8lDQ6AMj/idG0kOimA5.svg?c=1dxbfHSJFAPEGdCLU4o5B" alt="Snyk" height="44" /> Snyk OSS Vulnerabilities by Manifest

This repo demonstrates how to integrate **Snyk Open Source (OSS) vulnerability scanning** into GitHub Actions for a multi-language, multi-module environment. It includes workflows for:

- Java (Maven multi-module)
- Node.js (NPM)
- Python (Pip)
- Generic Template (no build steps added)

Each workflow:
- Scans dependencies using the Snyk CLI
- Parses the results to extract only **critical** and **high** severity issues
- Posts a formatted markdown comment to the **pull request**
- Writes the same summary to the **GitHub PR Checks tab**

> ⚠️ All workflows fetch the **latest Snyk CLI version** using [`snyk/actions/setup@master`](https://github.com/snyk/actions).
> ⚠️ Snyk advisor links are only supported for npm, PyPi, Go, and Docker (https://github.com/snyk/actions).

---

## 🔧 Setup Requirements

### 1. Add Snyk API Token
Create a repository secret named `SNYK_TOKEN`:
- Go to `Settings > Secrets and variables > Actions`
- Add a new secret with the name `SNYK_TOKEN`

---

### 2. Environment Setup per Language

| Language | Build Tool / Command                    | Required Setup                        |
|----------|------------------------------------------|----------------------------------------|
| Java     | `mvn clean install -DskipTests`          | `actions/setup-java@v3` with Java 8+  |
| Node.js  | `npm install`                            | `actions/setup-node@v3` with Node 16+ |
| Python   | `pip install -r requirements.txt`        | `actions/setup-python@v4` with Python 3.x |
| Generic  | None                                     | None (just `snyk test` on repo root)  |

---

## 📂 Workflows in This Repo

### ✅ `maven-multi-module-pr-comment.yml`
Scans all Maven modules using Snyk’s `--all-projects` and generates a PR comment summary.

- Uses `Temurin 8` to support legacy Java
- Builds all modules but skips tests
- Scans all `pom.xml` files
- Summarizes vulnerabilities grouped by manifest file
- Includes direct links to Snyk Advisor + Maven Central

---

### ✅ `npm-oss-pr-comment.yml`
Scans `package.json` dependencies for a Node.js project and posts a detailed markdown summary.

- Installs with `npm ci`
- Targets only high and critical vulns
- Includes CVSS, dependency type, fix info, and links

---

### ✅ `pip-oss-pr-comment.yml`
Scans Python dependencies via `requirements.txt`.

- Uses Python 3.x
- Installs dependencies via `pip install -r requirements.txt`
- Extracts critical/high issues with full context

---

### ✅ `snyk-generic.yml`
Runs a top-level Snyk scan against the entire repo (no language-specific config required).

- Useful for mono-repos or fallback scanning
- Doesn't require a build
- Posts vulnerabilities from all detectable ecosystems

---

## 💬 PR Comment Format

All workflows include a custom embedded Node.js script that:

- Summarizes findings into a markdown table
- Uses severity icons (🔴 critical, 🟠 high)
- Displays CVSS scores, fix availability, advisory links
- Differentiates between **Direct** and **Transitive** dependencies
- Automatically updates an existing PR comment using GitHub CLI

Example snippet:

```markdown
| Severity | Package                | Dependency Type | CVSS | Occurrences | Title                              | Fix Available | Advisor |
|----------|------------------------|------------------|------|-------------|------------------------------------|---------------|---------|
| 🔴 CRITICAL | log4j-core@2.14.1     | Transitive       | 9.8  | 3           | [RCE in log4j](https://snyk.io/...) | 2.17.1        | [View](https://security.snyk.io/...) |
