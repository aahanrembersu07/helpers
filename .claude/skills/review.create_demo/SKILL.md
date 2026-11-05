---
description: Write code_demo.md, a short demo of a tool: what to run and what it generates
model: sonnet
---

# Goal

- Given a dir that holds a tool, write the file `code_demo.md` in that dir
- The file shows a short demo of the tool in four parts:
  - What the tool does, in a few lines
  - **What to run**: the command, what it needs, what it costs, and how long it takes
  - **What is generated**: each output file, what it holds, and the stage that
    writes it
  - What a past run gave, and how to check the output
- Keep the file short: a reader must run the demo after reading it for 5 mins
- The file is the source of the end-to-end demo of `code_tour.md`
  - The skill `.claude/skills/review.create_code_tour/SKILL.md` points to this file
    and does not repeat it

# Conventions

- Write the file following the template
  `.claude/templates/review_create_demo.template.md`
- Write the text following `.claude/skills/text.rules.md` and
  `.claude/skills/markdown.rules.md`
- Name each file in backticks with the path from the repo root
- Put each command in a fenced `bash` block, with `>` before the command
- Take the facts from the README of the dir and from the files that a run leaves
  - Do not invent a command, a flag, a file, or a number
  - If a fact is not clear from the sources, write `unknown` and do not guess

# Workflow

## Get the Input

- Take the dir from the arguments
  - If no dir is given, use the current dir
  - If the dir does not exist, report the error and stop
- Write the output file in the dir, named `code_demo.md`
  - The name is fixed: it does not follow `## Generated File Naming` of
    `.claude/skills/skill.rules.md`
  - If the file exists, overwrite it after reading it
- Take the `README.md` of the dir
  - If it does not exist, report the error and stop
  - Tell the user to run `.claude/skills/readme_dir.create/SKILL.md` on the dir first

## Find the Demo

- Find the runner: the executable that runs all the stages, or the only executable
  - Take its path, its flags, and its example commands from the README of the dir and
    from the README of the script
  - Check the flags with `--help`
- Find the small example input
  - Take the smallest input file that the README or the tests use, e.g.,
    `paper_list.short.csv`
  - Skim all of the input file, so that the text says what the rows are
- Choose the output dir
  - Take it from the README when it gives one
  - Else use `./run_demo`, a dir in the current dir
- Find the setup that the command needs, e.g., `PYTHONPATH`, an API key, or Docker
  - Take it from the README of the dir
- Find what the demo needs and what it costs
  - Name the needs: internet, an LLM, a browser, an API key
  - Say that the demo costs LLM calls when it calls an LLM
  - Do not run a command that costs money: ask the user to run it
- Find the dry run, if the runner has `--dry_run`
  - Run it, and check that it needs no internet, no LLM, and no browser

## Find What Is Generated

- Take the list of the output files from the README of the dir, e.g., from the
  section "Output Dir"
- If the output dir of the demo exists, list it and compare it with the README
  ```bash
  > find <OUTPUT_DIR> -maxdepth 3 | sort
  ```
  - Keep only the files that the demo makes
  - If a file is in the output dir and not in the README, open it and add it
  - If a file is in the README and not in the output dir, say it is made only in
    some cases, and when
- For each file or group of files, write:
  - The name, with `<BASE>` or `NN` as in the README
  - What the file holds, in one short line: open the file if the README is not clear
  - The stage that writes it
- Cover the results, the issues, the summaries, and the cache and logs
- If the output dir exists, take the counts of a past run from the summary files
  - Take, for each stage, the items processed, completed, failed, and the issues
  - Take one or two values from a result file, e.g., a rating

## Write the File

- Fill the template, in this order:
  - `# What the Tool Does`: the purpose, the input, and the output, in 3 bullets
  - `# Run the Demo`: needs, cost, time, the command, and the dry run
  - `# What Is Generated`: the tree of the output dir, and the table of the files
  - `# Example Result`: the counts of a past run, one row per stage
  - `# Check the Output`: the checks that a reader can fail
- Write the table of the files with the columns `File`, `Written by`, and `Content`
- Write the example result only when an output dir exists
  - Say that it is the result of a past run
  - Say that the numbers can change with the internet and the LLM, but the files do
    not
- Write in `# Check the Output` these checks, when they apply:
  - Each summary file shows the counts of its stage
  - Each failed item is in the issues file of the same stage
  - A value of the report is equal to the value in the file of one item
  - A second run of the same command skips the finished work
- Do not add a diagram: the file is a short text

## Report

- Print the path of the file
- Print the demo command, what it needs, and what it costs
- Print the list of the files that the demo makes
- Do not commit the file

# Constraints

- Do not modify any file of the dir, other than `code_demo.md`
  - Running the scripts of the dir can create `*.log` files: delete those that the
    run created
- Do not run a command that needs internet, an LLM, or a browser without the user
- Do not invent a command, a flag, a file, or a number
- Use `:` and not `-` to separate a file from its description
- Do not use emojis or the em-dash

# Examples

- Run on the dir `research/Estimate_academic_paper_quality`
  - The runner is `estimate_paper_quality.py`, and the input is
    `paper_list.short.csv`
  - The command writes in `./run_demo`, and needs internet, a browser for SSRN
    papers, and an LLM
  - The output has one file per stage, e.g., `01-resolve.paper_list.json`, and one
    summary file per stage, e.g., `02-download.summary.txt`
  - A past run: 5 of 5 papers resolved, 3 of 5 downloaded, and 3 papers rated

# Verification

- [ ] The output file is named `code_demo.md` and is in the dir
- [ ] The five parts of the template are present, in order
- [ ] The command, the flags, and the input file exist, and the flags are in `--help`
- [ ] The file says what the demo needs, what it costs, and how long it takes
- [ ] The paths in the file are from the repo root
- [ ] Every file of the output dir that the demo makes is in the table of the files
- [ ] Every file in the table has its content and the stage that writes it
- [ ] The example result comes from the summary files of a past run, and says so
- [ ] The file has no diagram and no command that was invented
- [ ] No line is over 85 characters outside code blocks and tables
- [ ] No file of the dir changed, other than `code_demo.md`
