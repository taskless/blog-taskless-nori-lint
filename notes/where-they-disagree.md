# Where they disagree

After the 3 fixes in the git history, nori-lint and Taskless differ on 8 files across 4 rules. Each one was read by hand.

## `bold_italics`: one file only Taskless flags

`openai/skills` `.curated/yeet` writes *why*, *what* and *net change* with single underscores: `_why_`. nori-lint's scanner looks for `**`, `__` and `*`, so single-underscore italics get past it. Vale's markdown parser reads `_why_` as emphasis, the way any renderer does.

## `consecutive_blank_lines`: one file only nori-lint flags

`microsoft/skills` `azure-compliance` ends with one blank line, then the final newline: `...)\n\n`. nori-lint splits the file on `\n`, which produces an empty string after the last newline, and counts that as a second blank line. Its own fixer replaces `\n{3,}`, so `nori-lint fix` leaves the file unchanged while `nori-lint lint` keeps reporting it.

## `trailing_whitespace`: one file only nori-lint flags

`huggingface/skills` `huggingface-datasets` uses Windows line endings. nori-lint splits on `\n`, leaves the `\r` on each line, and `trimEnd()` removes it, so all 107 lines are reported. The file has no trailing spaces. For the same reason nori-lint can never see two blank lines in a row in a CRLF file, since each blank line is `"\r"` rather than `""`.

## `description_action`: 5 files only nori-lint flags

Two use a YAML block scalar:

```yaml
description: >-
  Use when the user asks about finding the best, top, or recommended model...
```

nori-lint reads the text after `description:` on the same line, which is `>-`, and reports that it doesn't start with "Use". `huggingface-best` and `m365-agent-evaluator` both start with "Use". The Taskless rule is scoped to `frontmatter.description`, so Vale parses the YAML first and checks the value.

Three have no frontmatter at all (`azure-app-onboard` `deploy`, `prepare` and `scaffold`). nori-lint reports the missing description here and again under `frontmatter`. The port reports it once, from `frontmatter-has-description`.
