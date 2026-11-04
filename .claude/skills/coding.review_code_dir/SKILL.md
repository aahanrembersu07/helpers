---
description: Write a leveled checklist to understand the code, docs, and tests of a dir
model: sonnet
---

# Goal

- Given a dir, write a markdown file with checkboxes that guides a human to read and
  understand the code, the documentation, and the tests of that dir
- Split the checklist in levels, from "Understand the code in 10 mins" to
  "Understand the code in full"
  - Each level adds items to the levels before it
  - A reader can stop after any level and still know the dir at that depth

# Conventions

- Write the checklist following the template
  `.claude/templates/coding_review_code_dir.template.md`
- Write the text following `.claude/skills/text.rules.md` and
  `.claude/skills/markdown.rules.md`
  - Use `- [ ]` for every checkbox, never a bare `[ ]`
- Refer to the Python rules in `.claude/skills/coding.rules.md` and to the test rules
  in `.claude/skills/testing.rules.md` in the items of Level 4

# Workflow

## Get the Input

- Take the dir from the arguments
  - If no dir is given, use the current dir
  - If the dir does not exist, report the error and stop
- Compute the path of the dir from the repo root:
  ```bash
  > git rev-parse --show-toplevel
  ```
- Write the output file in the dir, named
  `checklist-coding.review_code_dir.md`
  - This follows the convention in `.claude/skills/skill.rules.md`
    `## Generated File Naming`
  - If the file exists, overwrite it after reading it, and keep the items that are
    already ticked `- [x]` when the same item is still in the new checklist

## Take the Inventory

- List the tracked and the untracked files of the dir, without the ignored files:
  ```bash
  > git ls-files --cached --others --exclude-standard <DIR>
  ```
- Count the lines of every file with `wc -l`
- Mark the files that have uncommitted changes:
  ```bash
  > git status --short <DIR>
  ```
- Skip, and list in the "Skipped" line with the reason:
  - Logs: `*.log`
  - Scratch files: `tmp.*`
  - Caches: `__pycache__/`, `.ruff_cache/`, `.pytest_cache/`
  - Symlinks
  - Generated or big data files, e.g., a CSV over 1000 lines
    - Keep them in the table, but do not add an item to read them in full
    - A Level 4 item may list the header of the file

## Classify the Files

- Put each file in exactly one kind:
  - `doc`: `README.md`, `*.README.md`, files under `docs/`, `log.md`, `todos.md`
  - `test`: files under `test/` or named `test_*.py`
  - `code`: all the other `*.py` and `*.sh` files
  - `data`: prompts, YAML, CSV, JSON, and other inputs read by the code
- Find the entry points
  - A file is an entry point if it is executable, or has `_main` and
    `if __name__ == "__main__"`
- Pair each `code` file with:
  - Its doc, e.g., `<FILE>.README.md` or the section of a design doc that names it
  - Its test, e.g., `test/test_<FILE>.py`
- Find the test helpers, i.e., test files that have no `test_` prefix, e.g.,
  `test/epq_test_utils.py`

## Order the Code

- Build the import graph of the files of the dir
  - Read the `import` lines of each file, and keep the imports that point to
    another file of the dir
- Sort the `code` files bottom up
  - A file with no imports from the dir comes first
  - A file comes after all the files that it imports
  - Break ties with the smaller file first
  - Entry points come last, in the order that the docs give for running them
- If the graph has a cycle, report it in the "Map of the Dir" and sort the files of
  the cycle by size

## Understand Enough to Write Items

- For each file read the docstring or the first lines, and the doc that is paired
  with it
- Write the purpose of a file in one short line
  - If the purpose is not clear from the file, write only the file name and put
    `purpose unknown` in the line
  - Do not guess

## Write the Map

- Fill "Map of the Dir" of the template
  - One sentence for the purpose, from the README or the PRD
  - The entry points
  - The layers, bottom first, with one line per file
  - The table with one row per file, including the skipped data files
    - Write the paths in the table relative to the dir, and say so in the line
      above the table

## Write the Levels

- Write all four levels of the template
  - **Level 1: Understand the Code in 10 Mins**: what the dir does and how to run it
  - **Level 2: Understand the Code in 1 Hour**: components and data flow
  - **Level 3: Understand the Code in 4 Hours**: every code file and its test
  - **Level 4: Understand the Code in Full**: edge cases, diffs, rules, and gaps
- Fill each level with three groups of items: `Docs`, `Code`, `Tests`
  - In Level 3 use one group `Code and Tests` in place of `Code` and `Tests`
  - Add a fourth group `Check Yourself` with questions that the reader must answer
    without looking at the code
- Level 1 items
  - Read only the README summary and the entry point docstrings and arguments
  - Do not open a test body
  - Total size to read: under 300 lines
- Level 2 items
  - Read the architecture and the flow docs
  - Read the types file and `_main()` of each entry point, in the run order
  - Run the test suite once, and read the end-to-end test
  - If the dir has no types file, read the most imported `code` file
- Level 3 items
  - Write one item per `code` file, in the bottom up order, each followed by the item
    of its test file
  - If the test helpers were not read in Level 2, put them before the first test that
    uses them
  - In "Look at", name the main flow only, so that Level 4 can cover the private
    helpers and the error paths
  - Add an item to trace one input through all the stages
  - Add an item to check three claims in the docs against the code
- Level 4 items
  - Add every file that is not in the levels above, including `data` files
  - Add the private functions, the error paths, and the `TODO` comments
  - Add the `git diff` of the changed files when the inventory shows changes
  - Add the check against the coding rules and the testing rules
  - Add the coverage run
- Do not repeat an item in two levels
  - If a file is read in Level 3, its Level 4 item is about the part that Level 3
    skipped
- If a level has no item for a group, drop the group instead of writing a filler

## Write the Items

- Make each item one line of at most 85 characters, or two lines with the second
  line indented by 6 spaces
- Start each item with a verb: `Read`, `Run`, `List`, `Trace`, `Compare`, `Say`,
  `Name`, `Draw`
- Name each file in backticks with the path from the repo root
- For a `Read` item of a file, add the line count and `modified` when the file has
  uncommitted changes
- Add what to look at after the purpose
  - E.g., the classes, the public functions, the error handling, the files written
- Make each item checkable: the reader must know when it is done
  - **Bad** (no path from the repo root, no size, no purpose, no focus)
    ```markdown
    - [ ] Read epq_report.py
    ```
  - **Good**
    ```markdown
    - [ ] Read `research/Estimate_academic_paper_quality/epq_report.py` (535 lines,
          modified): writers and readers of the output files. Look at the issues file
    ```
  - **Bad** (a vague check)
    ```markdown
    - [ ] Understand the pipeline
    ```
  - **Good** (the reader can fail it)
    ```markdown
    - [ ] Name the four stages in order and the file that each one writes
    ```

## Report

- Print the path of the checklist file
- Print the number of items per level
- Print the list of skipped files, if any
- Do not commit the file

# Constraints

- Do not modify any file of the reviewed dir, other than the checklist file
- Do not invent a command, a file, or a function
  - Run the test command and the coverage command only if the dir shows how: a
    README, a `Makefile`, an `invoke` task, or a `pytest.ini`
  - If the command is unknown, write `find the test command` as the item text
- Do not put the whole code of the dir in the level 1 or level 2
- Use `:` and not `-` to separate a file from its description
- Do not use emojis or the em-dash

# Examples

- Run on the dir `research/Estimate_academic_paper_quality`
  - Level 1 reads `research/Estimate_academic_paper_quality/README.md` and
    `--help` of the four executables
  - Level 2 reads `docs/02-design.architecture.md`, `epq_types.py`, and the `_main()`
    of `resolve_papers.py`, `download_papers.py`, `extract_features.py`, and
    `report_papers.py`
  - Level 3 reads `epq_paths.py`, then `epq_issues.py`, and so on up to
    `estimate_paper_quality.py`, each followed by its test
  - Level 4 reads `qs_2027.csv` header, `extract_features.prompt.md`, and the diff of
    the modified files

# Verification

- [ ] The output file is named `checklist-coding.review_code_dir.md` and is in the
      reviewed dir
- [ ] The four levels are present, in order, with the titles of the template
- [ ] Every checkbox starts with `- [ ]` or `- [x]`
- [ ] Every `code`, `doc`, and `test` file of the inventory appears in at least one
      item, or in the "Skipped" line
- [ ] Every file named in the checklist exists
- [ ] The Level 3 code items follow the bottom up order of the import graph
- [ ] No item appears in two levels
- [ ] No line is over 85 characters outside code blocks
- [ ] No file of the reviewed dir changed, other than the checklist file
