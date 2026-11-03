---
description: Write or update a directory's README.md with structure, files, and executables
model: haiku
---

# Goal

- You are an expert technical writer specializing in software documentation.
- Given a directory path `<DIR>`, write a `README.md` in that directory following the
  conventions and structure in `.claude/skills/readme_dir.rules.md`

# Workflow

## Examine the Directory
- List all subdirectories, files (Python/Markdown), and executables

## Write the README
- Structure the README with sections from `.claude/skills/readme_dir.rules.md`:
   - Summary, as bold-labeled bullets (Goal, Input, Output, Users)
   - Structure of the Dir
   - Description of Files
   - Description of Executables (if applicable)
   - Output Dir (if the executables write a tree of files in one dir), as a `##`
     section of the executables, before the first executable
   - Description of Workflows (if applicable)
   - Description of Architecture (if applicable)

- Follow all formatting rules from `.claude/skills/readme_dir.rules.md`:
   - Description limits (e.g., <20 words for subdirs, 1-line for files)
   - Command formatting with `> ` prefix and proper code blocks
   - Example ordering (simple to complex)
   - Table format for command references
   - Bullets only, no prose paragraphs, including the intro of each section
   - Many outputs as one sub-bullet per file or per group of files

## Check the Commands
- Check each example command against the current CLI of its executable
  (e.g., with `--help`)
- Remove the args that the tool dropped, and rename the args that the tool renamed
- Check the commands in the workflows too

# Verification

- [ ] `README.md` exists in the target directory `<DIR>`
- [ ] Every required section from `.claude/skills/readme_dir.rules.md` is present
- [ ] The Summary has the bullets Goal, Input, Output, and Users
- [ ] There is an "Output Dir" section if the executables write a tree of files
- [ ] There is no prose paragraph
- [ ] Description length limits and command formatting rules are followed
- [ ] Every example command, in the executables and in the workflows, uses args that
  the current CLI accepts
