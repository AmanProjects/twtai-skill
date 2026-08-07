---
description: Scan a folder of Markdown for broken links, dead anchors, and duplicate headings. Reports file, line, and severity.
---

Run the documentation link checker over the user's docs and report what it finds.

## Input

The folder to scan is: $ARGUMENTS
If `$ARGUMENTS` is empty, scan the current working directory.

## Task

1. Run the bundled scanner:

   ```bash
   node "${CLAUDE_PLUGIN_ROOT}/scan.js" $ARGUMENTS
   ```

2. Read its output. It reports three kinds of finding:
   - **error** — a link points at a file that does not exist
   - **warning** — a link points at a `#anchor` with no matching heading in the target file
   - **info** — two headings share a slug, so the second one's anchor is silently numbered (`#setup-1`), and any link written to the plain anchor lands on the first

3. Summarise for the reader, grouped by severity, most severe first. For each finding give the file, the line, and what to change.

4. If there are no findings, say so in one line and stop.

## Constraints

- Do NOT edit any files unless the user asks you to fix the findings.
- Do NOT re-implement the check by reading files yourself — run the scanner.
- The scanner exits non-zero when it finds errors. That is expected; it makes the command usable in CI. Do not treat a non-zero exit as a failure of the command.

## Output

Lead with a one-line verdict, e.g. `3 broken links and 1 dead anchor across 42 files.`

Then a table:

| Severity | File | Line | Problem | Fix |
|---|---|---|---|---|

Close with the single highest-value fix if there is more than one finding.
