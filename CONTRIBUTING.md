# Contributing

Thanks for your interest in improving `/security-scan`! Contributions are welcome.

## How to contribute

1. Fork the repo
2. Create a branch (`git checkout -b improve-a03-checks`)
3. Edit `security-scan.md`
4. Test the skill in Claude Code (`/security-scan` against a real project)
5. Open a PR with a description of what you changed and why

## What makes a good contribution

- **New vulnerability patterns** — if you've seen a class of bugs that the current skill misses, add detection guidance for it under the relevant OWASP category.
- **Better remediation advice** — more specific fix instructions, especially for common frameworks (Express, Django, Rails, Spring, etc.).
- **Language-specific checks** — the skill is language-agnostic by design, but framework-specific guidance in the right places helps Claude give better results.
- **False positive reduction** — if the skill flags something that's almost always a non-issue, help tune the instructions to reduce noise.

## What to avoid

- Don't add checks that require external tools beyond what's in `allowed-tools`. The skill should work with a stock Claude Code install.
- Don't remove OWASP categories. Every A01–A10 category should remain covered.
- Don't change the output format without good reason — consistency matters for teams that parse the reports.

## Testing

The best way to test changes is to install the modified skill and run it against a project you know well:

```bash
cp security-scan.md ~/.claude/commands/
cd your-project
# In Claude Code:
/security-scan
```

Check that findings are accurate, severity ratings make sense, and remediation advice is actionable.
