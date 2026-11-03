# <Directory Name>

- **Goal**: <What this directory does, in one bullet>
- **Input**: <What the executables read>
- **Output**: <What the executables write>
  - <One sub-bullet per file or per group of files, if there are many>
- **Users**: <Who uses this directory and for what>

## Structure of the Dir

<List subdirectories if any: optional section if no subdirs>

- `<subdir1>/`, <Brief description of subdirectory (<20 words)>
- Example:
  ```markdown
  | File                                   | Description    |
  | -------------------------------------- | ---------------|
  | `ai.claude_code.how_to_guide_figs/`    | Screenshots and images for Claude Code setup and usage guide |
  | `ai.github_copilot_review.how_to_guide_figs/` | Screenshots demonstrating GitHub Copilot review workflow |
  ```

## Description of Files

- Python and Markdown files, alphabetical by path.

<List all Python and Markdown files in alphabetical order with 1-line
descriptions (<20 words)>

- `<file1.py>`, <One-line description of what this file does>
- Example:
  ```markdown
  | File                                   | Description                                                                  | Cluster             |
  | -------------------------------------- | ---------------------------------------------------------------------------- | ------------------- |
  | `bookmark_utils.py`                    | Shared helpers for downloading/uploading Google Sheets data and CSV files    | Shared Utilities    |
  | `download_academic_paper_to_md.py`     | Download an academic paper (arXiv/DOI/PDF), convert to Markdown, summarize   | Content Downloaders |
  | `download_to_md.py`                    | Detect input type and dispatch to the matching `download_*_to_md.py` script  | Content Downloaders |
  | `download_utils.py`                    | Shared helpers for fetching article titles and summarizing text via an LLM   | Shared Utilities    |
  | `podcast_dl.py`                        | Download and format a podcast transcript from various sources                | Podcast Tools       |
  ```

# Description of Executables

- There are <N> executables
  - <How they relate, e.g., they talk to each other only through files>

## Output Dir

<Optional section: add it when the executables write a tree of files in one dir>

- Every executable takes `--output_dir`, and the dir is the only place where files
  are written
  - <Who names the files>
  - <What a rerun skips>
- A file that a stage writes is named `<pattern>`, where <meaning of each part>
  ```text
  <output_dir>/
      <file_1>
      <file_2>
      <subdir>/
          <file_3>
  ```
- The files:

  | File       | Written by          | Content               |
  | :--------- | :------------------ | :-------------------- |
  | `<file_1>` | `<script_name.py>`  | <What the file holds> |
  | `<file_2>` | `<another_tool.sh>` | <What the file holds> |

- Why this layout
  - <One choice, with the link to its design decision>
    ([`D<N>`](/<path_from_repo_root>/docs/03-decisions.md#<anchor>))
- The names are in `<path_from_repo_root>/<names_file.py>`, and the rules are in
  `<path_to_rules_file>`

## `<script_name.py>`

### What It Does

<1-3 bullet points describing the tool's purpose>
- <Main functionality>
- <Key inputs/outputs>
- <Important side effects>

### Examples

<3-5 realistic usage patterns ordered from simple to complex>
<Use only the args that the tool accepts now>

- <Simple use case: short description>:
  ```bash
  > ./script_name.py --basic-arg value
  ```

- <Common workflow: short description>:
  ```bash
  > ./script_name.py --input file.txt --output result.txt
  ```

- <Advanced usage: short description>:
  ```bash
  > ./script_name.py --input file.txt --output result.txt --verbose --config custom.yaml
  ```

## `<another_tool.sh>`

### What It Does

<1-3 bullet points describing the tool's purpose>
- <Functionality>
- <Behavior>
- <Effect>

### Examples

- <First simple example>:
  ```bash
  > ./another_tool.sh input_dir
  ```

- <Second example with options>:
  ```bash
  > ./another_tool.sh input_dir --format csv
  ```

- <Complex example with multiple flags>:
  ```bash
  > ./another_tool.sh input_dir --format csv --recursive --output results.csv
  ```

# Description of Workflows

<Optional section describing how tools combine for complex features>

<Use bullet points to describe multi-tool workflows>
<Use only the args that each tool accepts now>

## <Workflow Name>

- **Purpose**: <What this workflow accomplishes>

- **Steps**:
  1. <Use tool A to prepare data>
     ```bash
     > tool_a.py --prepare input_file
     ```

  2. <Use tool B to process data>
     ```bash
     > tool_b.py --process prepared_file
     ```

  3. <Use tool C to analyze results>
     ```bash
     > tool_c.py --analyze processed_file
     ```
