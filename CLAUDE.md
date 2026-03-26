# Global CLAUDE.md — Simon Billinge

## Git Workflow (applies to all projects)

I use a **forking workflow**:
- `origin` = my personal fork
- `upstream` = canonical/group repo

### Branch discipline — ALWAYS follow this sequence:
1. `git checkout main`
2. `git pull upstream main`
3. `git checkout -b <feature-branch>` — keep branch names short, just enough to recall the topic (e.g. `fix-testpaths`, `feat-build-cli`, not `fix-configure-pytest-testpaths-to-prevent-collection`)
4. Work and commit on the feature branch
5. Push to `origin` with `git push -u origin <branch>` on first push (sets upstream tracking so subsequent pushes need only `git push`)
6. Open a PR from the fork into `upstream` (only when asked)

**Never commit directly to `main`.** Never branch off an existing branch unless explicitly instructed.

### Commit messages
Use conventional prefixes: `feat:`, `fix:`, `chore:`, `docs:`, `test:`, `refactor:`

## News Files (applies to all projects)

Every PR requires a news file. Create one immediately after branching using:

```
package add news <flag> "Brief user-facing summary (under 72 chars)"
```

Flags (mutually exclusive):
- `-a` — Added (new feature)
- `-c` — Changed (modification to existing feature)
- `-d` — Deprecated
- `-r` — Removed
- `-f` — Fixed (bug fix)
- `-s` — Security
- `-n` — No-news needed (trivial change; reviewer decides if that's appropriate)

The command reads the current branch name and creates `news/<branch-name>.rst`.
Stage and commit the news file along with the rest of the PR work.

For no-news, use: `package add news -n "reason why no news is needed"`

## Conda Environment Setup (applies to all projects)

When setting up or initialising a conda env for a project, install from **conda-forge** everything in:
- `requirements/conda.txt` (project-level)
- `requirements/tests.txt` (project-level)
- `~/dev.txt` (global, in the user's home directory)

```
micromamba install -n <env> -c conda-forge --file requirements/conda.txt --file requirements/tests.txt --file ~/dev.txt
```

Only fall back to pip for packages not on conda-forge.

## pre-commit (applies to all projects)

Run `pre-commit install` at the start of any project so hooks run automatically on every commit.

Never use `git commit --no-verify` (`-n`) except temporarily when auto-fixes need to be committed mid-cleanup. Once all issues are resolved, always commit without `-n`.

## Code Style (group standards)

### Docstrings
All docstrings must be in **NumPy format** (Parameters, Returns, etc. sections).

### Docstring parameter descriptions
All parameter descriptions in NumPy docstrings must begin with **"The"**.

### argparse help strings
Write argparse `help=` and `description=` strings to be as informative as possible. Include examples where helpful. Do **not** force them to start with "The" — some inputs are actions where that makes no sense.

### Exception message
All exception messages must include concise information about 'what is wrong' and 'how to fix it' when they are raised.

### File paths
Always use `pathlib.Path` for file and directory paths. Avoid `os.path`, string concatenation, or `open()` with raw strings.

### Code density and comments
- Prefer self-documenting code over explanatory comments. If a comment is needed, the code may need renaming.
- Only add blank lines where they semantically separate blocks with very different purposes. Do not add blank lines for visual breathing room.
- Follow PEP 8, but where PEP 8 is silent, lean compact. Three lines of clear code is better than the same code padded to six.
- This applies everywhere: source code, tests, config files.

### Testing
Use `pytest.mark.parametrize` for multiple cases. Each case is a tuple preceded
by a comment describing what is being tested and the expected outcome. Group all
success cases into one parametrized test and all failure cases into another:

```python
@pytest.mark.parametrize("inputs, expected", [
    (  # case 1: description — expected outcome
        [case1_inputs], case1_expected
    ),
    (  # case 2: ...
        [case2_inputs], case2_expected
    ),
])
def test_something_succeeds(inputs, expected):
    ...

@pytest.mark.parametrize("inputs, exc_match", [
    (  # case 1: bad input — raises ValueError
        [bad_inputs], "error message fragment"
    ),
])
def test_something_raises(inputs, exc_match):
    with pytest.raises(ValueError, match=exc_match):
        ...
```

### Formatting
- Black (line length 79)
- isort (black profile)
- flake8 (max line 79, ignore E203)
- docformatter (PEP 257 compliant)
