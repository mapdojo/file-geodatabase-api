# Encoding And Line Endings

## File Encoding

Use UTF-8 without BOM for source, project, solution, markdown, JSON, XML, YAML, PowerShell, and script files unless a tool or framework explicitly requires another encoding.

Do not add UTF-8 BOM to new files. If an editor reports a file loaded with the wrong encoding, fix the file encoding rather than accepting noisy encoding metadata changes.

## Editor Metadata

Keep editor encoding metadata broad and minimal. Do not add per-file encoding entries for normal source files unless the file genuinely needs a non-standard encoding.

Expected default:

```text
UTF-8 without BOM
```

Accept per-file encoding entries only for deliberate legacy files, generated files, or external files that cannot safely be converted.

## Line Endings

Follow the repository's existing line-ending behavior and `.gitattributes` rules. Avoid mechanical line-ending-only changes in feature commits.

If a file must be rewritten to normalize encoding or line endings, keep that change isolated in a dedicated commit so reviewers can separate formatting churn from behavior changes.

## Cleanup Checklist

- [ ] Scan for unexpected BOM or per-file encoding metadata before committing broad file moves.
- [ ] Convert normal source files to UTF-8 without BOM.
- [ ] Avoid changing unrelated line endings while editing docs or source.
- [ ] Run `git diff --check` before committing.
- [ ] Keep encoding-only cleanup in its own commit.
