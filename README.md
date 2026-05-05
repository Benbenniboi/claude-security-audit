# /security-scan

A Claude Code slash command that runs a deep, OWASP Top 10 security audit on your codebase. It uses Opus for maximum reasoning depth and produces a CVSS-scored report with actionable remediation steps.

This isn't a linter or regex matcher — it's a full semantic security review. Claude reads your code, understands the context, and identifies real vulnerabilities the way a senior security engineer would.

## What it does

The skill runs a 4-phase audit:

**Phase 1 — Reconnaissance:** Reads your project manifests, maps the tech stack, identifies the attack surface (routes, APIs, auth boundaries, data stores), and checks git history for recently changed security-sensitive files.

**Phase 2 — Vulnerability Scan:** Analyzes the entire codebase against all 10 OWASP Top 10 (2021) categories — injection, broken access control, cryptographic failures, auth bypass, SSRF, misconfigurations, and more.

**Phase 3 — Secrets Scan:** Grep-based scan for hardcoded API keys, tokens, passwords, and credentials across all common file types.

**Phase 4 — Report:** Produces severity-ranked findings (Critical → Low) with OWASP categories, file locations, proof snippets, impact analysis, CVSS scores, and a prioritized remediation checklist.

## Install

### Per-project (recommended)

```bash
mkdir -p .claude/commands
cp security-scan.md .claude/commands/
```

Commit `.claude/commands/security-scan.md` to your repo so your whole team gets it.

### Global (all projects)

```bash
mkdir -p ~/.claude/commands
cp security-scan.md ~/.claude/commands/
```

### One-liner

```bash
# Project
mkdir -p .claude/commands && curl -sL https://raw.githubusercontent.com/benscheibe/security-scan/main/security-scan.md -o .claude/commands/security-scan.md

# Global
mkdir -p ~/.claude/commands && curl -sL https://raw.githubusercontent.com/benscheibe/security-scan/main/security-scan.md -o ~/.claude/commands/security-scan.md
```

## Usage

```
/security-scan                    # Full codebase audit
/security-scan src/api/           # Audit a specific directory
/security-scan server.js          # Audit a single file
/security-scan src/auth/login.ts  # Focus on a specific module
```

## Example output

```
### 🔴 CRITICAL (Severity: 9.0-10.0)

**1. SQL Injection in User Lookup**
- **OWASP Category:** A03:2021 — Injection
- **File:** `src/api/users.ts:47`
- **Description:** User-controlled input interpolated directly into SQL query
- **Proof:** `db.query(`SELECT * FROM users WHERE id = ${req.params.id}`)`
- **Impact:** Full database read/write, data exfiltration, auth bypass
- **Remediation:** Use parameterized queries: `db.query('SELECT * FROM users WHERE id = ?', [req.params.id])`
- **CVSS Score:** 9.8 (network exploitable, no auth required, full confidentiality/integrity impact)

### ✅ What's Good
- HTTPS enforced via HSTS header
- Passwords hashed with bcrypt (cost factor 12)
- CSRF tokens validated on all state-changing endpoints

### Summary Scorecard
| Category                     | Findings | Highest Severity |
|------------------------------|----------|-----------------|
| A01 Broken Access Control    | 2        | HIGH            |
| A03 Injection                | 1        | CRITICAL        |
| A07 Auth Failures            | 1        | MEDIUM          |
| **Total**                    | **4**    | **CRITICAL**    |
```

## Configuration

The skill uses these defaults in its frontmatter:

| Setting | Value | Why |
|---------|-------|-----|
| `model` | `claude-opus-4-7` | Security analysis benefits from maximum reasoning depth |
| `allowed-tools` | `Read, Grep, Glob, Bash(grep:*), Bash(find:*), Bash(cat:*), Bash(git diff:*), Bash(git log:*), Bash(npm audit:*), Bash(pip audit:*), Bash(cargo audit:*)` | Scoped to read-only operations + package audits |
| `argument-hint` | `[file-or-directory]` | Optional — scopes the scan to a specific path |

To override the model (e.g., use Sonnet for faster/cheaper scans), edit the `model:` line in the frontmatter:

```yaml
model: claude-sonnet-4-6
```

## What it checks

| OWASP Category | Examples |
|---------------|----------|
| A01 Broken Access Control | IDOR, missing auth checks, CORS misconfig, path traversal |
| A02 Cryptographic Failures | Hardcoded secrets, weak hashing (MD5/SHA1), `Math.random()` for tokens |
| A03 Injection | SQLi, XSS, command injection, NoSQL injection, SSTI, XXE |
| A04 Insecure Design | Missing input validation, race conditions, business logic flaws |
| A05 Security Misconfiguration | Debug mode in prod, default creds, missing security headers |
| A06 Vulnerable Components | Known CVEs via `npm audit` / `pip audit` / `cargo audit` |
| A07 Auth Failures | Weak passwords, no brute-force protection, JWT issues |
| A08 Data Integrity Failures | Unsafe deserialization, missing SRI, CI/CD vulnerabilities |
| A09 Logging Failures | Logging PII/tokens, missing audit trails, leaked stack traces |
| A10 SSRF | User-controlled URLs in server-side requests, DNS rebinding |

## Language support

The skill is language-agnostic — it works with any codebase Claude can read. The dependency audit step auto-detects your package manager:

- **Node.js** → `npm audit`
- **Python** → `pip audit`
- **Rust** → `cargo audit`

If your language isn't covered by the auto-detect, Claude still performs the full code analysis — just without the automated dependency CVE check.

## Tips

- **Run after auth changes.** Any time you touch login flows, JWT handling, session management, or access control, run `/security-scan src/auth/` to catch regressions.
- **Pair with `/review`.** Use `/review` for general code quality, then `/security-scan` for a focused security deep-dive.
- **CI integration.** Add the scan to your PR workflow with [claude-code-security-review](https://github.com/anthropics/claude-code-security-review) for automated checks on every pull request.
- **Scope it down.** Full codebase scans on large repos use a lot of tokens. Target specific directories when you know where the changes are.

## Requirements

- [Claude Code](https://claude.ai/code) v2.1.0+
- Claude Pro, Team, or Enterprise plan (or API key with Opus access)

## License

MIT — see [LICENSE](LICENSE).
