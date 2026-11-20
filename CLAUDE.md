# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repository is

Jupyter Book (v1, `jupyter-book<2.0`) documentation for the **teareduce**
Python package. The package source lives in a sibling repository
(`../teareduce`, https://github.com/nicocardiel/teareduce). The built site is
deployed at https://nicocardiel.github.io/teareduce-cookbook/intro.html.

Changes to documented behaviour usually involve both repositories: the API is
defined in `../teareduce/src/teareduce/` (public names are re-exported in
`__init__.py`; examples use `import teareduce as tea`), and the cookbook only
demonstrates it.

## Build

```console
$ pip install -r requirements-dev.txt   # local: adds teareduce[cleanest] + PyCosmic
$ jupyter-book clean .
$ jupyter-book build .
$ open _build/html/index.html
```

`requirements.txt` holds only what the CI build needs; `requirements-dev.txt`
adds the heavy dependencies required to actually run the notebooks.

## Deployment and notebook execution

- `.github/workflows/documentation.yaml` builds on every push to `main`
  (Python 3.12, `pip install -r requirements.txt`) and publishes `_build/html/`
  to the `gh-pages` branch.
- `_config.yml` sets `execute_notebooks: off`: notebooks are **never executed**
  during the build, locally or in CI. What appears on the website is exactly the
  outputs stored in the committed `.ipynb` files. After editing a notebook (or
  after a change in teareduce that affects its output), re-run it locally and
  commit it with outputs. The CI environment does not have the `cleanest`
  extras, so switching back to `cache`/`force` would break notebooks that need
  them.
- `allow_errors: True` means a failing cell does not stop the build; errors are
  rendered on the page, so check outputs before committing.
- `only_build_toc_files: true`: a new page is ignored unless it is added to
  `_toc.yml`.

## Content layout

- `notebooks/<topic>/<topic>.ipynb`: one tutorial notebook per topic, with its
  input FITS/data files in the same directory. Notebooks fetch these inputs with
  `tea.get_cookbook_file('notebooks/<topic>/<file>')`, which downloads from
  `raw.githubusercontent.com/nicocardiel/teareduce-cookbook/main/...` into the
  current directory (skipping existing files). A new input file therefore has to
  be committed and pushed to `main` before the download works for readers.
- Files produced when running notebooks (e.g. intermediate `c*ftdz_*.fits`,
  `w*ftdz_*.fits`, PDFs in `notebooks/wavecalib/`) are listed in `.gitignore`;
  add new generated outputs there rather than committing them.
- `docs/cleanest/`: MyST Markdown page for the `tea-cleanest` script, with
  screenshots in `images/`. Its `--help` listing is generated, not hand-written:
  from `docs/cleanest/files/` run `../../extract_ansi_help.sh tea-cleanest`
  (see `run_extract_blocks_ansi_help.txt`). This captures the coloured
  `rich-argparse` output into `help_tea-cleanest.md` as an
  `ansi-shell-session` code block, which `cleanest.md` pulls in with
  `{include}`. Regenerate it whenever the script's arguments change in
  teareduce.
- Citations use `{cite}` with keys from `references.bib` (ADS bibcodes as keys,
  `author_year` style); `bibliography.md` renders the list.

## Conventions

- Shell examples use the `(venv_tea) $ ` prompt inside ```` ```console ````
  blocks; `_config.yml` configures sphinx-copybutton to strip that prompt (and
  a bare `$ `). Blocks that should have no copy button get the class
  `no-copybutton` (styles in `_static/custom.css`).
- Text is written in English.
