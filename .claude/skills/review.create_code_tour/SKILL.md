---
description: Write code_tour.md, a leveled checklist to understand the code, docs, and tests of a dir
model: sonnet
---

# Goal

- Given a dir, write the file `code_tour.md`: a markdown file with checkboxes that
  guides a human to read and understand the code, the documentation, and the tests of
  that dir
- Split the checklist in exactly three levels:
  - Level 1: "Understand in 15 mins"
  - Level 2: "Understand in 1 hour"
  - Level 3: "Understand in full"
  - Do not add a fourth level, and do not rename a level
  - Each level adds items to the levels before it
  - A reader can stop after any level and still know the dir at that depth
- Give the time of every item, so that the reader can plan the work
- Include examples and tests to run, so that the reader sees how the code behaves
- Include the end-to-end demo in Level 3, taken from the file `code_demo.md` of the
  dir
  - The skill `.claude/skills/review.create_demo/SKILL.md` writes this file
  - The tour does not repeat the explanation of the output: it points to the file
- Take the files, their kinds, the time to read each one, and the order of the code
  from the section `# Review roadmap` of the `README.md` of the dir
  - The skill `.claude/skills/readme_dir.create/SKILL.md` writes this section
  - The checklist does not repeat the map or the table of the files

# Conventions

- Write the checklist following the template
  `.claude/templates/review_create_code_tour.template.md`
  - The template has no `# Map of the Dir`: the map is in the README
- Write the text following `.claude/skills/text.rules.md` and
  `.claude/skills/markdown.rules.md`
  - Use `- [ ]` for every checkbox in a list, never a bare `[ ]`
- Refer to the Python rules in `.claude/skills/coding.rules.md` and to the test rules
  in `.claude/skills/testing.rules.md` in the items of Level 3
- Take the way to run the tests from the subsection `## Testing` of the roadmap

# Workflow

## Get the Input

- Take the dir from the arguments
  - If no dir is given, use the current dir
  - If the dir does not exist, report the error and stop
- Compute the path of the dir from the repo root:
  ```bash
  > git rev-parse --show-toplevel
  ```
- Write the output file in the dir, named `code_tour.md`
  - The name is fixed: it does not follow `## Generated File Naming` of
    `.claude/skills/skill.rules.md`
  - If the file exists, overwrite it after reading it, and keep the items that are
    already ticked when the same item is still in the new checklist
- Take the `README.md` of the dir
  - If it does not exist, or it has no section `# Review roadmap`, report the error
    and stop
  - Tell the user to run `.claude/skills/readme_dir.create/SKILL.md` on the dir first
- Take the `code_demo.md` of the dir
  - If it does not exist, report the error and stop
  - Tell the user to run `.claude/skills/review.create_demo/SKILL.md` on the dir
    first

## Read the Roadmap

- Read the section `# Review roadmap` of `<DIR>/README.md`
- From the bullets of the section take:
  - The entry points, in run order
  - The layers, bottom first, and the cycles in the import graph
  - The command to run the tests, in the dev env and in Docker
  - The reading speeds, in lines per min
  - The "Skipped" line, and the other notes on the table
- From the table take, for each file: the kind, the lines, the `Mins`, the `Changed`
  flag, its doc, and its test
  - The order of the `code` rows is the bottom up order of the import graph
  - A raw export has no time: give it no read item
  - A Level 3 item may list the header of a raw export
- Check that the roadmap is current:
  ```bash
  > git ls-files --cached --others --exclude-standard <DIR>
  > wc -l <FILES>
  ```
  - If a file is missing from the table, or a `Lines` value differs, report it and
    stop
  - Tell the user to run `.claude/skills/readme_dir.create/SKILL.md` on the dir again
- Mark the files that have uncommitted changes, since the `Changed` flag goes stale
  first:
  ```bash
  > git status --short <DIR>
  ```

## Set the Reading Time

- Use the reading speeds of the roadmap
  - The time of a `Skim` item uses the skim speed
  - The time of a `Read` item uses the speed of the kind of the file
- Compute the minutes of an item as `ceil(lines / speed)`, with 1 as the minimum
- Estimate the other items by hand, in whole mins:
  - Run `--help` or a dry run: 2
  - Run `pytest` on one test file: 1
  - Run `pytest` on the whole dir: 2, or the measured time rounded up
  - Run a command that needs the internet, an LLM, or a browser: 5 to 30, and say
    what it needs
  - Check a claim of the docs against the code: 5
  - Run the coverage: 5
  - Check a file against a rules file: 2 per file
- Show the time at the start of each item as `(<N> mins)`, and `(1 min)` for one min
- Sum the times of the items of each level
  - Print the sum in the `Time` line of the level
  - For Level 2 and the next, also print the sum up to that level
  - Use these fixed titles, and do not compute a title from the sums:
    - `# Level 1: Understand in 15 Mins`
    - `# Level 2: Understand in 1 Hour`
    - `# Level 3: Understand in Full`
  - The `Time` line holds the real sum, so the reader can see the gap with the title

## Understand Enough to Write Items

- For each file read the docstring or the first lines, and the doc that is paired
  with it
- Write the purpose of a file in one short line
  - Take the purpose of the entry points and of the layers from the roadmap
  - If the purpose is not clear from the file, write only the file name and put
    `purpose unknown` in the line
  - Do not guess
- Find the docs that contain a diagram, e.g., mermaid, PlantUML, C4, Graphviz, or
  TikZ:
  ```bash
  > grep -rlE '```(mermaid|plantuml|dot|graphviz|tikz)|C4Context|C4Container|C4Component|@startuml' <DIR> --include='*.md'
  ```
  - Note the kind of the diagram of each file, e.g., `C4 diagram (PlantUML)`
- Find the examples to run
  - Take the example commands from the README of the dir and of the scripts
  - Take the flags from `--help`, e.g., `--dry_run` and `-v DEBUG`
  - Run each command that needs no internet, no LLM, and no browser, and check
    that it works
  - Do not invent a command or a flag

## Write the Levels

- Write all three levels of the template
  - **Level 1**: what the dir does and how to run it
  - **Level 2**: components and data flow
  - **Level 3**: every file, its test, the checks, and the end-to-end demo
- Fill each level with the groups `Docs`, `Code`, `Tests`, and `Examples`
  - In Level 3 use one group `Code and Tests, Bottom Layer First` in place of `Code`
    and `Tests`, and add the groups `Checks` and `End-to-End Demo`
  - If a level has no item for a group, drop the group instead of writing a filler
- Read or skim whole files only
  - Start each reading item with `Read all of` or `Skim all of`
  - Do not limit an item to a section, a function, or a range of lines
  - Do not write `Look at`, section names, or function names as the scope of an item
  - The only exception is the header of a raw export
- Use `Skim all of` to map a file, and `Read all of` to understand it
  - A file may appear in two levels only when the second item goes deeper, i.e.,
    from `Skim all of` to `Read all of`
  - Level 3 reads every file in full, so a file appears in it once, with
    `Read all of`: do not skim a file and then read it in Level 3
- Level 1 items
  - Skim the README of the dir, which holds the `# Review roadmap`, and the README of
    its docs
  - Skim the small example input
  - Run `--help` of the entry points
  - Run the dry run of the runner and of the first stage
  - List the test files, and match each one with the code it tests, using the table
    of the roadmap
  - Do not open a code file or a test file
  - Keep the level under 20 mins
- Level 2 items
  - Read the architecture doc, and skim the flow doc
  - Skim the types file and each stage
  - Run all the tests with `pytest`, and in Docker
  - Run the end-to-end test with the live log
  - Read the end-to-end test, and skim the test helpers
  - Keep the level under 90 mins
- Level 3 items
  - In `Docs`, read the flow doc, the PRD, the decisions, the progress doc, and the
    plan
  - In `Docs`, read each design doc and each script README right before the code that
    it describes, in the order of the code
  - In `Docs`, read the logs, the todos, the background notes, and the `data` files
    that the code uses, e.g., config, prompt, and example input
  - In `Code and Tests`, write for each `code` file, in the order of the `code` rows
    of the roadmap, i.e., bottom up:
    - `Read all of` the file
    - `Read all of` its test file, if it has one
    - `Run pytest` on its test file and match each test name with the code
  - If the test helpers were not read in Level 2, or were only skimmed, put
    `Read all of` them before the first test that uses them
  - In `Checks`, add:
    - Three items that check a claim of the docs against the code
    - An item to mark each doc line that disagrees with the code with
      `// TODO(gp): Check ...`
    - The `git diff` of the changed files when `git status` shows uncommitted
      changes, and skip it when the dir is clean
    - The check against the coding rules and the testing rules
    - The coverage run
  - In `Examples`, run the dry run again with `-v DEBUG`
  - In `Examples`, run the first stage on the example input, and run the dry run of
    the next stages on its output
    - Say what it needs, e.g., `needs internet`
  - In `Examples`, add an item to trace one input through all the stages with the
    end-to-end test
  - In `End-to-End Demo`, add these items, in this order:
    - `Read all of` the file `code_demo.md`
    - Run the demo with the command of `code_demo.md`: say what it needs and what it
      costs, and copy the command, the needs, and the cost from the file
    - List the output dir, and match each file with the section `# What Is Generated`
      of `code_demo.md`
    - Read each summary file, and check that each failed item is in the issues file
    - Compare two values of the report with the value in the file of one item
    - Run the same command again, and check that the finished work is skipped

## Write the Items

- Start each item with its time, then the checkbox text: `- [ ] (<N> mins) ...`
- Make each item at most 4 lines of at most 85 characters, with the continuation
  lines indented by 6 spaces
  - Split an item that needs more lines into two items
  - A line that holds only one path may pass 85 characters
- Start each action with a verb: `Read all of`, `Skim all of`, `Run`, `List`,
  `Check`
- Name each file in backticks with the path from the repo root
- For each doc that contains a diagram, add a render step to its reading item, so
  that the reader sees the diagram and not its source
  - Keep `Read all of` or `Skim all of` at the start of the item, and add the line
    `It has a <KIND> diagram: render it first:`
  - Add under it the command in a fenced block, with the path from the repo root:
    ```bash
    > open_md.py -i <DOC_PATH>
    ```
  - Do this in every item that reads or skims the doc, and add no time for it
  - Do not use `open_md.py -i` for a doc with no diagram
- For a reading item, add the line count and `modified` when the file has uncommitted
  changes, then the purpose
- For a `Run` item, put the command in a fenced block under the item
  - Use `pytest <PATH>` or `invoke docker_cmd --cmd "pytest <PATH>"` for tests
  - Do not use `uv` to run the tests
- Make each item checkable: the reader must know when it is done
  - **Bad** (no time, no path from the repo root, part of a file, no purpose)
    ```markdown
    - [ ] Read the functions `_parse()` and `_main()` of epq_report.py
    ```
  - **Good**
    ```markdown
    - [ ] (27 mins) Read all of `research/Estimate_academic_paper_quality/epq_report.py`
          (535 lines): writers and readers of the output files
    ```
  - **Bad** (a vague check)
    ```markdown
    - [ ] (5 mins) Understand the output dir
    ```
  - **Good** (the reader can fail it)
    ```markdown
    - [ ] (5 mins) Check claim 1: each file in the tree of "Output Dir" has a
          `get_*_path` function in `research/Estimate_academic_paper_quality/epq_paths.py`
    ```

## Report

- Print the path of the checklist file
- Print the number of items and the time of each level
- Print the list of skipped files of the roadmap, if any
- Do not commit the file

# Constraints

- Do not modify any file of the reviewed dir, other than the checklist file
  - Running the scripts of the dir can create `*.log` files: delete those that the
    run created
- Do not invent a command, a file, or a function
  - If the command is unknown, write `find the command` as the item text
- Use `:` and not `-` to separate a file from its description
- Do not use emojis or the em-dash

# Examples

- Run on the dir `research/Estimate_academic_paper_quality`
  - Level 1 skims `research/Estimate_academic_paper_quality/README.md`, and runs the
    dry run of `estimate_paper_quality.py` on `paper_list.short.csv`
  - Level 2 reads the architecture doc, skims `epq_types.py` and the four stages, and
    runs the end-to-end test with the live log
  - Level 3 reads `epq_paths.py`, reads `test/test_epq_paths.py`, runs it, and so on
    up to `populate_affiliation_tiers.py`, then reads the plan and the decisions
  - Level 3 ends with the end-to-end demo: it reads `code_demo.md`, runs the demo
    into `./run_demo`, matches each output file with `code_demo.md`, and checks the
    summary of each stage

# Verification

- [ ] The output file is named `code_tour.md` and is in the reviewed dir
- [ ] Exactly three levels are present, in order, with the fixed titles
      `Understand in 15 Mins`, `Understand in 1 Hour`, and `Understand in Full`
- [ ] Every checkbox starts with `- [ ]` or `- [x]`
- [ ] Every item starts with `(<N> mins)` or `(1 min)` after the checkbox
- [ ] The `Time` line of each level equals the sum of the times of its items
- [ ] The group `End-to-End Demo` is in Level 3, with the items that read
      `code_demo.md`, run the demo, and check the output
- [ ] The group `End-to-End Demo` has no explanation of the output files: it points
      to `code_demo.md`
- [ ] The checklist has no `# Map of the Dir`, no reading speeds, and no table of the
      files
- [ ] The header says that the map and the table are in the section
      `# Review roadmap` of the `README.md` of the dir
- [ ] The paths in the items are from the repo root
- [ ] The time of each reading item follows the reading speeds of the roadmap
- [ ] Every item that reads a file starts with `Read all of` or `Skim all of`
  - The only exceptions are the diff items and the header of a raw export
- [ ] No reading item contains `Look at`, a section name, or a function name as its
      scope
- [ ] Every `code`, `doc`, and `test` file of the table of the roadmap appears in at
      least one item, or in the "Skipped" line of the roadmap
- [ ] Every file named in the checklist exists
- [ ] The Level 3 code items follow the order of the `code` rows of the roadmap,
      i.e., bottom up in the import graph
- [ ] No item appears twice with the same action in two levels
- [ ] No file is skimmed and then read in Level 3
- [ ] Every item that reads a doc with a diagram has the step
      `open_md.py -i <DOC_PATH>`, and no other item has it
- [ ] Every example command was run, or says what it needs
- [ ] No test command uses `uv`
- [ ] No line is over 85 characters outside code blocks, tables, and lines that
      hold only one path
- [ ] No item is longer than 4 lines
- [ ] No file of the reviewed dir changed, other than the checklist file
