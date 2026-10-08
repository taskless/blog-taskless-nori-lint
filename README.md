# blog-taskless-nori-lint

[nori-lint](https://github.com/tilework-tech/nori-lint) is an opinionated linter for `SKILL.md` files, written by the team at [Nori](https://noriagentic.com) and released under Apache-2.0. Its rules come from a clear point of view: a skill is read by an LLM, every wasted word costs context in every session, and a skill should be a short process with a checklist.

This repo rebuilds nori-lint's 12 static rules as [Taskless](https://taskless.io) rules, so they run in `taskless check` next to a team's other rules, on every commit and in CI. It then runs both tools over 365 public `SKILL.md` files and lines up what each one flags. What we learned along the way, including what didn't work, is in [`notes/`](notes/).

nori-lint's 10 LLM rules (`obvious_instructions`, `negative_without_positive` and the rest) aren't ported. They ask a model for a judgment, and that's what nori-lint is for.

## Run it

You need Node 22.22 or later.

```sh
npm install
npm run corpus     # fetch every SKILL.md from the repos in corpus/repos.txt, at the pinned commits
npm run compare    # nori-lint and Taskless over corpus/skills/, side by side
npm run test:rules # the rules' own fixtures
```

`node scripts/compare.mjs --diff <rule>` lists the files the two tools disagree on for one nori-lint rule. nori-lint 0.4.0 and Taskless 0.12.0 are pinned in `package.json`.

## The rules

All 15 are Vale rules in `.taskless/rules/vale/`, scoped to `**/SKILL.md`. 13 are declarative, and 2 are short Tengo scripts, because they need to count or to know whether a line sits inside a code fence.

| nori-lint rule | Taskless rule | How |
| --- | --- | --- |
| `bold_italics` | `bold-italics` | `existence` over Vale's `strong` and `emphasis` scopes |
| `consecutive_blank_lines` | `consecutive-blank-lines` | `existence`, raw |
| `description_action` | `description-action` | `existence` over `frontmatter.description` |
| `frontmatter` | `frontmatter-has-name`, `frontmatter-has-description`, `frontmatter-known-fields`, `frontmatter-description-length` | `occurrence` with `min: 1` for the required fields, `existence` for the rest |
| `frontmatter_name_format` | `frontmatter-name-format` | `existence` over `frontmatter.name` |
| `line_count` | `line-count` | `occurrence` with `max: 150` |
| `markdown_links` | `markdown-links` | `script` |
| `redundant_title` | `redundant-title` | `existence`, raw, anchored to the start of the file |
| `required_tags` | `required-tags` | `occurrence` with `min: 1` |
| `trailing_whitespace` | `trailing-whitespace` | `existence`, raw |
| `unclosed_tags` | `unclosed-tags` | `script` |
| `when_to_use` | `when-to-use` | `existence` over `heading` |

Two checks inside nori-lint's `frontmatter` rule aren't ported: `compatibility` over 500 characters, and `metadata` that isn't a map. Neither fires anywhere in the corpus.

## Scorecard

Over 365 `SKILL.md` files from nine repositories, by file:

| nori-lint rule | Files nori-lint flags | Files Taskless flags | Both | nori-lint only | Taskless only |
| --- | --- | --- | --- | --- | --- |
| `bold_italics` | 311 | 312 | 311 | 0 | 1 |
| `consecutive_blank_lines` | 29 | 28 | 28 | 1 | 0 |
| `description_action` | 324 | 319 | 319 | 5 | 0 |
| `frontmatter` | 40 | 40 | 40 | 0 | 0 |
| `frontmatter_name_format` | 11 | 11 | 11 | 0 | 0 |
| `line_count` | 239 | 239 | 239 | 0 | 0 |
| `markdown_links` | 169 | 169 | 169 | 0 | 0 |
| `redundant_title` | 358 | 358 | 358 | 0 | 0 |
| `required_tags` | 362 | 362 | 362 | 0 | 0 |
| `trailing_whitespace` | 90 | 89 | 89 | 1 | 0 |
| `unclosed_tags` | 257 | 257 | 257 | 0 | 0 |
| `when_to_use` | 15 | 15 | 15 | 0 | 0 |

8 rules agree on every file. The 8 files where they differ are explained one by one in [`notes/where-they-disagree.md`](notes/where-they-disagree.md). In each, the Taskless result matches what the file actually contains.

## The corpus

`corpus/repos.txt` pins each repository to a commit. `corpus/fetch.sh` keeps only the `SKILL.md` files and writes them to `corpus/skills/`, which is gitignored, since the files belong to their authors.

| Repository | Skills |
| --- | --- |
| anthropics/skills | 20 |
| anthropics/claude-code | 10 |
| anthropics/claude-plugins-official | 33 |
| obra/superpowers | 15 |
| openai/skills | 44 |
| microsoft/skills | 205 |
| huggingface/skills | 26 |
| vercel-labs/agent-skills | 9 |
| tilework-tech/nori-skillsets | 3 |

251 of the 365 live under a dot-directory such as `.github/skills/` or `.curated/`. `scripts/compare.mjs` runs nori-lint through its library export, once per file, so they're all included. See [`notes/dot-directories.md`](notes/dot-directories.md) for why that matters.

## Credit

The rules, their thresholds and the reasoning behind them are nori-lint's. The two scripts follow its scanning logic. nori-lint is Copyright Tilework Tech and licensed under the [Apache License 2.0](https://github.com/tilework-tech/nori-lint/blob/main/LICENSE). This repo is MIT.
