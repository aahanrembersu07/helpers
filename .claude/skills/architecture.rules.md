This file contains conventions for software architecture and code organization

# ARCH1: Layers and Dependencies

## `ARCH1.1`: Organize Code in Layers

- Split code into layers, each with one responsibility:
  - **Script layer**: parse CLI arguments (`_parse()`)
  - **Orchestration layer**: call the other layers in order (`_main()`)
  - **Business logic layer**: algorithms and data transformations, no I/O
  - **Infrastructure layer**: file I/O, system calls, databases, external APIs
  - **Utilities layer**: generic helpers and data structures
- Point dependencies in one direction only:
  - The script layer calls the orchestration layer
  - The orchestration layer calls business logic and infrastructure
  - Business logic and infrastructure call only utilities
- Treat business logic and infrastructure as siblings: neither calls the other
- Pass data from infrastructure to business logic through function arguments
- Accept data, not paths, in business logic
  - E.g., `analyze_dataset(df: pd.DataFrame)`, not `analyze_dataset(path: str)`
- This enables:
  - Testing business logic without I/O
  - Swapping a data source or file format
  - Reusing business logic in a different script
- Test business logic with plain inputs and test infrastructure with real files
  - See `.claude/skills/testing.rules.md`
- **Bad** (file I/O and printing are mixed with the computation)
  ```python
  def process_data(input_file: str) -> None:
      with open(input_file) as f:
          data = json.load(f)
      result = complex_calculation(data)
      print(result)
  ```
- **Good** (one function per layer, `_main()` calls them in order)
  ```python
  def _read_input(input_file: str) -> Dict[str, Any]:
      """
      Infrastructure layer: read the input file.
      """
      with open(input_file) as f:
          return json.load(f)


  def process_data(data: Dict[str, Any]) -> Dict[str, Any]:
      """
      Business logic layer: compute the result.
      """
      return complex_calculation(data)


  def _write_output(result: Dict[str, Any], output_file: str) -> None:
      """
      Infrastructure layer: write the result.
      """
      with open(output_file, "w") as f:
          json.dump(result, f)


  def _main(parser: argparse.ArgumentParser) -> None:
      """
      Orchestration layer: call the other layers in order.
      """
      args = parser.parse_args()
      data = _read_input(args.input_file)
      result = process_data(data)
      _write_output(result, args.output_file)
  ```

## `ARCH1.2`: Keep Interfaces Between Layers Minimal

- Pass only the data that the callee needs, not a whole context object
- Do not add callbacks from an inner layer to an outer layer
- Do not add bidirectional dependencies between two layers
- **Bad** (the callee depends on the structure of `context` and changes it)
  ```python
  def process(context: Any) -> None:
      context.data = transform(context.data)
      context.log(f"Processed {context.count} items")
      context.state = "done"
  ```
- **Good** (the callee gets one input and returns one output)
  ```python
  def process(data: pd.DataFrame) -> pd.DataFrame:
      """
      Transform the data.
      """
      return transform(data)


  # The caller manages logging and state.
  processed = process(data)
  ```

## `ARCH1.3`: Enforce Dependency Direction

- Do not create import cycles between modules
- Do not import a module of a higher layer from a module of a lower layer
- Put types shared by several modules (dataclasses, enums, constants) in one leaf
  module that imports no other module of the project
- Check the direction with a unit test that parses the imports of every module
  and fails on a forbidden import
- **Bad** (`report.py` imports `pipeline.py`, which already imports `report.py`)
  ```python
  # report.py
  import pipeline

  def render(df: pd.DataFrame) -> str:
      return df.to_string(max_rows=pipeline.MAX_ROWS)
  ```
- **Good** (both modules import the leaf module `project_types.py`)
  ```python
  # report.py
  import project_types

  def render(df: pd.DataFrame) -> str:
      return df.to_string(max_rows=project_types.MAX_ROWS)
  ```

## `ARCH1.4`: Split Modules by Responsibility

- Group related functions in one module
- Give each module one responsibility and name it after that responsibility
- In the helpers library, name modules `h<module>` (e.g., `hdownload.py`)
- Outside the helpers library, follow the naming of the surrounding project
- **Bad** (unrelated functions in one module)
  ```python
  # hutils.py
  def download_data() -> pd.DataFrame:
      ...

  def train_model(df: pd.DataFrame) -> Model:
      ...

  def generate_report(model: Model) -> str:
      ...
  ```
- **Good** (one module per responsibility)
  ```python
  # hdownload.py: data fetching.
  def download_data() -> pd.DataFrame:
      ...

  # hmodel.py: ML training.
  def train_model(df: pd.DataFrame) -> Model:
      ...

  # hreport.py: report generation.
  def generate_report(model: Model) -> str:
      ...
  ```

# ARCH2: Data Flow and Interfaces

## `ARCH2.1`: Pass Data Through Arguments and Return Values

- Make every data dependency explicit in the function signature
- Use type hints to show what data flows between functions
- Document the expected format in the docstring
  - E.g., the required columns of a `pd.DataFrame`
- Never use a global variable to pass data between functions
- **Bad** (stages share hidden state, so `stage2()` works only after `stage1()`)
  ```python
  _data: Optional[pd.DataFrame] = None

  def stage1() -> None:
      global _data
      _data = fetch()

  def stage2() -> None:
      global _data
      _data = transform(_data)
  ```
- **Good** (data flows through arguments and return values)
  ```python
  def stage1() -> pd.DataFrame:
      """
      Fetch the data from the source.

      :return: DataFrame with columns 'feature', 'target'
      """
      return fetch()


  def stage2(data: pd.DataFrame) -> pd.DataFrame:
      """
      Transform the fetched data.

      :param data: DataFrame with columns 'feature', 'target'
      :return: DataFrame with the added column 'prediction'
      """
      return transform(data)


  # In the orchestration layer.
  data = stage1()
  data = stage2(data)
  ```

## `ARCH2.2`: Bundle Parameters Into a Config Dataclass

- Use a config dataclass when a function has more than 3 parameters that tune its
  behavior
- Keep the required inputs in the signature and move the options to the config
- Make config fields required, unless the rule
  `## Use Default Values Rarely and Force Keyword-Only Via `*`` in
  `.claude/skills/coding.rules.md` allows a default
- Declare in the config WHAT to do, and implement HOW in the code
  - Do not store expressions or code in config values
- **Bad** (too many options in the signature, even if keyword-only)
  ```python
  def transform(
      data: pd.DataFrame,
      *,
      scale: bool,
      normalize: bool,
      remove_outliers: bool,
      handle_missing: str,
      log_transform: bool,
  ) -> pd.DataFrame:
      ...
  ```
- **Good** (the config groups the related options)
  ```python
  @dataclass
  class TransformConfig:
      scale: bool
      normalize: bool
      remove_outliers: bool
      handle_missing: str
      log_transform: bool


  def transform(data: pd.DataFrame, config: TransformConfig) -> pd.DataFrame:
      """
      Apply the transformations selected in the config.
      """
      ...
  ```
- **Bad** (the config embeds logic as a string)
  ```python
  config = {"transform": "if value > 0 then log(value) else 0"}
  ```
- **Good** (the config declares the intent and the code implements it)
  ```python
  @dataclass
  class Config:
      transform_type: str  # "log" or "none".


  if config.transform_type == "log":
      result = log_transform(data)
  ```

## `ARCH2.3`: Return and Pass Dataclasses, Not Tuples

- Return a dataclass or a `NamedTuple` when a function returns 3 or more values,
  or values with different meanings
- Allow a plain tuple only for a pair of values of the same kind
  - E.g., `(train_df, test_df)`
- Group inputs that always travel together in one dataclass
- **Bad** (the order of the 4 returned values is implicit)
  ```python
  def analyze(data: pd.Series) -> Tuple[float, float, int, float]:
      return mean, std, count, median
  ```
- **Good** (the fields are named)
  ```python
  @dataclass
  class Statistics:
      mean: float
      std: float
      count: int
      median: float


  def analyze(data: pd.Series) -> Statistics:
      """
      Compute the summary statistics of the data.

      :param data: input values
      :return: mean, std, count, and median of the values
      """
      return Statistics(mean=..., std=..., count=..., median=...)
  ```
- **Bad** (the relation between the 3 lists is implicit)
  ```python
  def analyze(
      values: List[float],
      indices: List[int],
      timestamps: List[datetime],
  ) -> Any:
      ...
  ```
- **Good** (one object owns the related data)
  ```python
  @dataclass
  class TimeSeries:
      values: List[float]
      indices: List[int]
      timestamps: List[datetime]


  def analyze(series: TimeSeries) -> Statistics:
      ...
  ```

## `ARCH2.4`: Do Not Modify Inputs

- Do not modify an argument in place: return a new object
- This prevents bugs from shared mutable state
- Allow an in-place function only when a copy costs too much (e.g., a huge
  `pd.DataFrame`)
  - End its name with `_inplace`
  - State in the docstring that it modifies the input
- **Bad** (the function changes the caller's DataFrame)
  ```python
  def normalize(df: pd.DataFrame) -> pd.DataFrame:
      """
      Normalize the DataFrame in place.
      """
      value = df["value"]
      df["value"] = (value - value.mean()) / value.std()
      return df
  ```
- **Good** (the function returns a normalized copy)
  ```python
  def normalize(df: pd.DataFrame) -> pd.DataFrame:
      """
      Return a normalized copy of the DataFrame.
      """
      df_norm = df.copy()
      value = df_norm["value"]
      df_norm["value"] = (value - value.mean()) / value.std()
      return df_norm
  ```

# ARCH3: Pipelines

## `ARCH3.1`: Split Sequential Processing Into Stages

- Organize processing with 3 or more sequential steps as a pipeline of stages
- Make each stage:
  - Have one responsibility
  - Take its input as an argument and return its output
  - Be idempotent: running it twice is safe and gives the same result
  - Depend on other stages only through its input
- This enables:
  - Testing each stage alone
  - Adding, removing, or reordering stages
- Pass data by return value only when all the stages always run together
  - When users can select stages, see `ARCH3.3`
- **Bad** (one function mixes all the stages)
  ```python
  def run() -> None:
      data = fetch_from_api()
      data = transform(data)
      save_to_storage(data)
      remove_temp_files()
  ```
- **Good** (one function per stage, the orchestration passes the data)
  ```python
  def _download() -> pd.DataFrame:
      """
      Download the data from the source.
      """
      return fetch_from_api()


  def _process(data: pd.DataFrame) -> pd.DataFrame:
      """
      Transform and validate the data.
      """
      return transform(data)


  def _upload(data: pd.DataFrame) -> None:
      """
      Upload the processed data to the destination.
      """
      save_to_storage(data)


  # In the orchestration layer.
  data = _download()
  data = _process(data)
  _upload(data)
  ```

## `ARCH3.2`: Select Stages With hselect_action

- Use `helpers.hselect_action` so that users can run a subset of the stages
  - The flags are `--action`, `--skip_action`, `--all_actions`, `--clear_actions`
  - See `.claude/skills/script.rules.md` `### Action Selection`
- Define each action as a private function `_<action_name>()`
  - E.g., the action `"download"` runs `_download()`
- Define `_VALID_ACTIONS` (every stage) and `_DEFAULT_ACTIONS` (stages that run
  by default) at the top of the script
- List the valid actions in the order they run, and document that order
- Register the actions in `_parse()` and run them in `_main()`
- **Bad** (no way to skip or select a stage)
  ```python
  def _main(parser: argparse.ArgumentParser) -> None:
      args = parser.parse_args()
      _download()
      _process()
      _upload()
      _cleanup()
  ```
- **Good** (the user selects the stages)
  ```python
  import helpers.hselect_action as hselacti

  _VALID_ACTIONS = ["download", "process", "upload", "cleanup"]
  _DEFAULT_ACTIONS = ["download", "process", "upload"]


  def _parse() -> argparse.ArgumentParser:
      parser = argparse.ArgumentParser(...)
      hselacti.add_action_arg(parser, _VALID_ACTIONS, _DEFAULT_ACTIONS)
      return parser


  def _main(parser: argparse.ArgumentParser) -> None:
      args = parser.parse_args()
      actions = hselacti.select_actions(
          args, _VALID_ACTIONS, _DEFAULT_ACTIONS
      )
      while actions:
          action = actions[0]
          to_execute, actions = hselacti.mark_action(action, actions)
          if not to_execute:
              continue
          if action == "download":
              _download(args.raw_file)
          elif action == "process":
              _process(args.raw_file, args.processed_file)
          elif action == "upload":
              _upload(args.processed_file)
          elif action == "cleanup":
              _cleanup(args.raw_file, args.processed_file)
  ```

## `ARCH3.3`: Pass Data Between Selectable Stages Through Files

- Assume that a user can skip any stage that `hselect_action` makes selectable
- Do not rely on the return value of another stage
- Make each stage read its input from a file and write its output to a file
  - Take the file paths from the CLI arguments
- This enables running one stage alone on the output of an earlier run
- **Bad** (`--skip_action download` leaves `data` undefined)
  ```python
  if action == "download":
      data = _download()
  elif action == "process":
      data = _process(data)
  ```
- **Good** (each stage reads and writes a file, so any stage runs alone)
  ```python
  if action == "download":
      _download(args.raw_file)
  elif action == "process":
      _process(args.raw_file, args.processed_file)
  ```

## `ARCH3.4`: Split a Pipeline Into Executables

- Split a pipeline into one executable per stage when any of these holds:
  - Stages have different dependencies
    - E.g., one stage needs a GPU, another needs only `pandas`
  - Stages have different runtimes
    - E.g., one stage takes hours, another takes seconds
  - Stages run on different schedules
    - E.g., download every day, report every week
  - The single script is too large to read or test
- Make the executables communicate only through files on disk
  - Read the input from a file and write the output to a file
  - Pass the file paths as command line arguments
  - Do not share state between executables through imports or global variables
- This enables:
  - Re-running one stage without re-running the others
  - Running stages on different machines or containers
  - Inspecting the intermediate results
- Keep a single script with `hselect_action` when all of these hold:
  - Stages share the same dependencies
  - Stages have similar runtimes and run on the same schedule
  - The script stays small enough to read and test
- Use `hselect_action` inside an executable for its own internal stages
- **Bad** (one script mixes stages with different dependencies and schedules)
  ```bash
  > run_pipeline.py --all_actions
  ```
- **Good** (one executable per stage, linked by files on disk)
  ```bash
  > download_data.py --output_file tmp.raw.csv
  > train_model.py --input_file tmp.raw.csv --output_file tmp.model.pkl
  > generate_report.py --input_file tmp.model.pkl --output_file tmp.report.md
  ```

## `ARCH3.5`: Sequence Executables With a Driver Script

- Write a driver script when users run the same 3 or more executables in the same
  order
- Write the driver in bash first: `run_<flow>.sh`
  - Follow `.claude/skills/bash.rules.md`
- Convert the driver to Python (`run_<flow>.py`) when any of these holds:
  - Users need to run only some of the steps through the driver
  - The driver needs an option other than the directory
  - The driver must forward `-v` or `--dry_run` to the executables
  - The driver needs a conditional or a loop, not only a straight sequence
  - The driver has more than 50 lines
- Keep the executables runnable alone: the driver is a convenience, not a
  requirement
- Put only sequencing in the driver: no business logic
- Take one directory as argument and derive every intermediate file path from it
- Stop at the first failing step
  - E.g., `set -e` in bash
- In the Python driver:
  - Call each executable with `hsystem.system()`
  - Forward `-v` and `--dry_run` to each executable
    - See `# Standard Argument Helpers` and `# Dry Run` in
      `.claude/skills/script.rules.md`
  - Use `hselect_action` with one action per executable, named after it
    - See `ARCH3.2`
- **Bad** (the user types each command and repeats the paths by hand)
  ```bash
  > download_data.py --output_file tmp.raw.csv
  > train_model.py --input_file tmp.raw.csv --output_file tmp.model.pkl
  > generate_report.py --input_file tmp.model.pkl --output_file tmp.report.md
  ```
- **Good** (a bash driver: one command, paths derived from one directory)
  ```bash
  #!/bin/bash
  # """
  # Run the flow: download the data, train the model, and generate the report.
  #
  # Usage: run_flow.sh <DIR>
  # """

  set -e

  DIR=$1
  mkdir -p "$DIR"

  # Download the data.
  download_data.py --output_file "$DIR/raw.csv"
  # Train the model.
  train_model.py --input_file "$DIR/raw.csv" --output_file "$DIR/model.pkl"
  # Generate the report.
  generate_report.py --input_file "$DIR/model.pkl" --output_file "$DIR/report.md"
  ```
- **Good** (after conversion to Python, a user runs one step alone)
  ```bash
  > run_flow.py --dir tmp.flow
  > run_flow.py --dir tmp.flow --clear_actions --action train_model
  ```

# ARCH4: Validation

## `ARCH4.1`: Validate at the Boundary

- Validate user input and option combinations in `_main()`, right after parsing
  and before any work starts
- Use `hdbg.dassert_*` with a message, not `if ... raise`
  - See `.claude/skills/coding.rules.md` `# Error Handling and Assertions`
- Do not repeat the same check of user input in the inner layers
- Keep a cheap `dassert` in an inner layer only for an invariant that, if broken,
  gives a wrong result without an error
- **Bad** (the same check in every layer)
  ```python
  def download(url: str) -> bytes:
      hdbg.dassert_ne(url, "", "URL required")
      ...


  def fetch(url: str) -> bytes:
      hdbg.dassert_ne(url, "", "URL required")
      ...
  ```
- **Bad** (an invalid option combination fails deep in the processing)
  ```python
  def process(config: Config) -> None:
      hdbg.dassert(
          not config.output_dir or config.format,
          "--format is required when --output_dir is specified",
      )
  ```
- **Good** (`_main()` validates once and the inner layers trust the input)
  ```python
  def _main(parser: argparse.ArgumentParser) -> None:
      """
      Orchestration layer: validate at the boundary, then delegate.
      """
      args = parser.parse_args()
      hdbg.dassert_ne(args.url, "", "--url must not be empty")
      hdbg.dassert(
          not args.output_dir or args.format,
          "--format is required when --output_dir is specified",
      )
      data = download(args.url)
      ...
  ```
