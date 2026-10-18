# Changelog

All notable changes to AMiGO are listed in this file.

## [0.2.0] - 2026-10-18

AMiGO is now a regular Python package that is installed with pip. The
configuration moved from the files in `defaults/` to environment variables, and
the `amigo` command replaces the shell alias. The calculations themselves are
unchanged: the tutorials give identical results.

### Upgrading from 0.1.0

**Installation**
- Install AMiGO with `pip install .` from the repository, instead of adding
  the repository to `PYTHONPATH`. The `PYTHONPATH` entry and the
  `alias amigo=...` line from the old instructions are no longer needed;
  remove the alias, because it takes precedence over the new `amigo` command.
- GYRE (v9.x or newer) still only has to be available as source code; it does
  not have to be compiled.

**`defaults/config.dat` is no longer read.** Set environment variables instead:

| 0.1.0 (`defaults/config.dat`) | 0.2.0 (environment)                                  |
|-------------------------------|------------------------------------------------------|
| `gyre_dir = /path/to/gyre`    | `export GYRE_DIR=/path/to/gyre`                      |
| `nthreads = 8`                | `export AMIGO_NTHREADS=8` (or `OMP_NUM_THREADS=8`)   |

Without `GYRE_DIR`, AMiGO stops with an error that says so. Without
`AMIGO_NTHREADS` or `OMP_NUM_THREADS`, the `amigo` command uses a single thread.
The variable can also be set for one run: `AMIGO_NTHREADS=8 amigo my_inlist.dat`.

**Running AMiGO on an inlist**

```
# 0.1.0
$ python <dir>/amigo/amigo/compute_rotation.py my_inlist.dat
# 0.2.0
$ amigo my_inlist.dat
```

**Creating a new inlist.** `defaults/amigo_inlist-blueprint.dat` is gone. Use
`amigo init my_inlist.dat` instead. Unlike the old blueprint, the new inlist
contains no placeholders that break the parser: the numerical settings are the
AMiGO defaults and the other values are examples. Set the star name and the
pattern file, and adapt the parameter ranges to the star.

**Python scripts.** Function calls that worked in 0.1.0 still work, with the
same arguments and return values. Adjust your scripts only in these cases:
- *Scripts that read `defaults/config.dat`*, e.g. a copy of the
  `read_config_file()` helper from tutorials 4 to 7, fail because the file is
  gone. `gyre_dir` can now be left out of `gravity_modes()`, which then uses
  `$GYRE_DIR`:

  ```python
  # 0.1.0
  gyre_dir, nthreads = read_config_file()
  mode = gravity_modes(gyre_dir, kval=0, mval=1)

  # 0.2.0
  mode = gravity_modes(kval=0, mval=1)    # uses $GYRE_DIR
  mode = gravity_modes(gyre_dir, kval=0, mval=1)    # still works, overrides $GYRE_DIR
  ```

- *`nthreads` in library calls* (`fit_with_grid`, `calculate_centrifugal_deformation`,
  ...) is unchanged: the functions use the value you pass or their own default,
  and ignore the environment variables. Pass it explicitly where you used the
  value from `config.dat`.
- *`read_inlist()`* returns the same values as before, but takes `gyre_dir` and
  `nthreads` from the environment. `gyre_dir` is `None`, rather than `''`, when
  `GYRE_DIR` is not set.

**Inlists**
- Existing inlists keep working.
- `output_dir`: a directory that does not exist is now created, and the results
  are saved in it. In 0.1.0 nothing was saved in that case and the figures were
  shown instead. Use `output_dir = ''` to only show the results, or
  `interactive = True` to save and show them.
- `True` and `False` have to be written exactly like that: other spellings
  that 0.1.0 accepted, such as `.True.`, now give an error.

### Added
- The `amigo` command: `amigo <inlist>` models the observed pattern(s),
  `amigo init <inlist>` creates a new inlist, and `amigo -h` shows the usage.
- `amigo.compute_rotation.init_inlist()`, the function behind `amigo init`.
- The environment variables `GYRE_DIR`, `AMIGO_NTHREADS` and `OMP_NUM_THREADS`.
- Defaults for the optional `&output` settings (`interactive = False`,
  `output_dir = '.'`): an inlist without them no longer fails with a `KeyError`.
- Inlist values in exponent notation, such as `5e-3`.

### Changed
- The default numerical settings are built in (`NUMERICAL_DEFAULTS` in
  `amigo.compute_rotation`) instead of being read from
  `defaults/default_numerical.dat`. The values are unchanged.
- `gravity_modes()`: `gyre_dir` is optional and defaults to `$GYRE_DIR`.
- The inlist blueprint ships with the package and is a valid inlist.
- A missing `output_dir` is created (see above).
- The `amigo` command only suppresses numerical `RuntimeWarning`s, instead of
  all warnings.
- With a single thread, the calculations run without a process pool, which
  makes small fits several times faster.
- The installation instructions in `README.md` and in the LaTeX source
  (`doc/latex-source.zip`) describe the new setup.
  `doc/Amigo-documentation.pdf` has not been rebuilt yet and still describes
  0.1.0.

### Fixed
- Multiprocessing pools are closed after use (no more
  `ResourceWarning: unclosed running multiprocessing pool`).
- Compatibility with scipy 1.14 and newer, which removed `cumtrapz`.
- GYRE's `src/tar/` directory is added to `sys.path` only once, instead of
  once per mode identification.
- Ctrl-C (`KeyboardInterrupt`) is no longer swallowed by the fallback error
  handlers in the fits and the centrifugal-deformation calculation.
- With AMiGO installed, running a script from the directory that contains the
  repository no longer breaks `import amigo`.

### Removed
- The `defaults/` directory: `config.dat` (replaced by environment variables),
  `default_numerical.dat` (built in) and `amigo_inlist-blueprint.dat` (moved
  into the package, see `amigo init`).
- The `__init__.py` in the repository root.
- The `read_config_file()` helpers in tutorials 4 to 7.

## [0.1.0]

The code before it became an installable package (up to commit `918c642`):
run as a script through `PYTHONPATH` and a shell alias, and configured with
the files in `defaults/`.
