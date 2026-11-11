# How to Document Research

## Summary

- Goal: a reader sees what was run, what result came out, what was asked and decided,
  and what to do next, without asking
- Use this guide for a project that runs experiments, e.g., in notebooks
- The implementation, the findings, and the decisions on the implementation go in
  `progress.md`
  - See
    [`docs/planning/all.plan_work.how_to_guide.md`](/docs/planning/all.plan_work.how_to_guide.md#keep-a-progress-log)
- The visual evidence (e.g., screenshots of the results), the research questions,
  their discussion, and the research decisions go in a Google Doc (Gdoc)
  - GitHub is only for code, and gets a link to each research question
  - See
    [Keep Research Discussions in the Gdoc](#keep-research-discussions-in-the-gdoc)
- Keep the log and the paper updated
  - See [Keep the Log and the Paper Updated](#keep-the-log-and-the-paper-updated)
- Control the research through the paper: a PR that changes a result, a model, or an
  assumption says which part of the paper or the Gdoc it changes
  - See
    [Control the Research Through the Paper](#control-the-research-through-the-paper)

## Keep the Log and the Paper Updated

- Keep the log and the paper updated
  - They are how we keep track of the results of the research
- Split the content by level of detail:
  - The notebooks hold all the details
  - The log (`progress.md`) holds all the experiments
  - The paper holds the interesting experiments

## Keep a Google Doc Per Project

- Keep one Google Doc for the research of each project
  - The Google Doc holds the visual evidence, the research questions, their
    discussion, and the research decisions
  - `progress.md` holds the findings and the decisions made to implement the
    mechanisms
- Organize the Google Doc so that each result is easy to find and to link:
  - A results tab, with an H1 per type of experiment and an H2 per sub-experiment
  - For each experiment: the date, the issue and the PR, the notebook and its cell,
    the `config` of the run, the figures, a few bullets on what it shows, and its
    decisions
  - A tab for the agenda of the meetings, with the open research questions
- Write each experiment so that a reader gets the gist without running the code, and
  the author can reproduce it from its `config`
- Save the Google Doc in the shared
  [research dir](https://drive.google.com/drive/folders/1XQ5zrk80eScbcynOsc1loDppzAjGNy2x)
  - Create a subdir named exactly like the GH issue of the project
  - Put the Google Doc and its supporting files in that subdir
- Link the Google Doc from the GH issue, the PR, and `progress.md`

## Keep Research Discussions in the Gdoc

- The rules:
  1. Stuff in GitHub and in Slack is meant to be lost
  2. GitHub is only for coding questions; research questions go in the Gdoc
  3. All the research discussions happen in the Gdoc, with comments and suggestion
     mode, and GitHub gets only a link to the question
- Where each thing lives:

  | Place                 | Holds                                                      | Never holds          |
  | :-------------------- | :--------------------------------------------------------- | :------------------- |
  | Gdoc                  | Results, research questions, their threads and decisions   | Code discussion      |
  | Repo                  | Code, notebooks, tests, the plan, `progress.md`, the paper | Research threads     |
  | GitHub issues and PRs | Tasks, coding questions, code review, links to the Gdoc    | Research discussion  |
  | Slack                 | Pings, with the link to the Gdoc                           | Questions, decisions |

- The life of a research question:
  1. Ask it in the Gdoc, as a comment that @mentions who should answer: on the result
     it is about, or on its line in the list of open questions
     - One comment per question: the context in one line, the question, the options,
       and a recommendation
     - Propose a change of the text in suggestion mode
  2. Link it from GitHub: one line in the issue, with the link to the Gdoc heading or
     comment, and no discussion
  3. Answer it in the comment thread
  4. Decide: a "Decided" bullet under the experiment, with who decided and when, then
     resolve the thread
  5. Record it in the repo: the entry of `progress.md` for the PR that acts on it
     links to the decision, and a change of scope goes in the plan
- Do not write code that depends on an open research question
- When an answer comes on GitHub or in a meeting, reply in the Gdoc thread with the
  answer and its link, record the decision, and resolve the thread
- An AI agent can draft the questions and the text of the Gdoc, but a human posts
  them; an agent never answers a research question, accepts a suggestion, or resolves
  a thread

## Control the Research Through the Paper

- A PR is hard to review by itself, so the reviewer compares what it means with the
  paper or the Gdoc
  - Every PR that changes a result, a model, or an assumption names the paper section
    or the Gdoc heading that it changes, and updates it: the paper in the PR itself,
    the Gdoc when the PR is opened
  - So the reviewer reviews the change of the paper or of the Gdoc, more than the
    code
- Such a PR has this section in its description, with each item ticked once it is
  answered:

  ```markdown
  ## Paper and Gdoc

  - [ ] Affected: <paper section and Gdoc heading, with links>
  - [ ] Results: <each result, figure, or number that changes, old -> new>, or none
  - [ ] Updated to match: <the paper in this PR; the Gdoc heading, with the date>
  - [ ] Assumptions: <each new or changed one, with its source>, or none
  ```

- Any other PR says so in one line, e.g., "Paper and Gdoc: no change, tests only"
- A PR that changes results with no change to the paper, `progress.md`, or the Gdoc
  is a red flag, and the checklist makes it visible
- Mark a value or a shape with no source as an assumption, e.g., "our assumption", so
  that a reader can tell it from a cited one

## Make the Research Reproducible

- Add unit tests for the notebooks
  - Fast tests as smoke tests, with a small run
  - Slow and superslow tests to freeze important research results
  - See `.claude/skills/notebook.rules.md` section "Testing Notebook"
- Add unit tests for the code with the same approach
  - See `.claude/skills/testing.rules.md`
- Use a `config` (e.g., a Python dict) that controls all the parameters of an
  experiment
  - The `config` becomes the signature of the experiment
  - Given the `config`, one can reproduce exactly the set up

## Write One Entry Per Day of Work

- Write one entry for every day of work with:
  - Screenshots of the results from the notebooks
  - Pointers to the relevant sections and files
    - E.g., the notebook, the section header, the script
  - Comments on what each experiment means
  - What was learned
  - What is next
- Use this skeleton for each daily entry:

  ```text
  Date: <date>

  Screenshots:
  - <Screenshot of a result from a notebook>
    - Where: <notebook / section / file>
    - What it does: <what the code or the plot does>

  Learned:
  - <What was learned from the results>

  Next:
  - <The next step>
  ```

## Example

- Use the
  [research Gdoc](https://docs.google.com/document/d/19KGM-1a9U5E7CrE4ltljYz7IVtMfNuRzrqgGkkuXUSQ/edit)
  as the model
  - A "Research results" tab with one Heading 1 per experiment
  - A "Project" tab with the open research questions and their answers
