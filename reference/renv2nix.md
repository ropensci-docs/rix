# Convert an `renv.lock` File to a Nix Expression

Convert an `renv.lock` File to a Nix Expression

## Usage

``` r
renv2nix(
  renv_lock_path = "renv.lock",
  project_path,
  return_rix_call = FALSE,
  method = c("fast", "accurate"),
  override_r_ver = NULL,
  ...
)
```

## Arguments

- renv_lock_path:

  Character, path of the renv.lock file, defaults to "renv.lock"

- project_path:

  Character, where to write `default.nix`, for example
  "/home/path/to/project". The file will thus be written to the file
  "/home/path/to/project/default.nix". If the folder does not exist, it
  will be created.

- return_rix_call:

  Logical, return the generated rix function call instead of evaluating
  it this is for debugging purposes, defaults to `FALSE`

- method:

  Character, the method of generating a nix environment from an
  renv.lock file. "fast" is an inexact conversion which simply extracts
  the R version and a list of all the packages in an renv.lock file and
  adds them to the `r_pkgs` argument of
  [`rix()`](https://docs.ropensci.org/rix/reference/rix.md). This will
  use a snapshot of `nixpkgs` that should contain package versions that
  are not too different from the ones defined in the `renv.lock` file.
  For packages installed from GitHub or similar, an attempt is made to
  handle them and pass them to the `git_pkgs` argument of
  [`rix()`](https://docs.ropensci.org/rix/reference/rix.md). Currently
  defaults to "fast", "accurate" is not yet implemented.

- override_r_ver:

  Character, defaults to NULL, override the R version defined in the
  `renv.lock` file with another version. This is especially useful if
  the `renv.lock` file lists a version of R not (yet) available through
  Nix, or if the R version included in the `renv.lock` is too old
  compared to the package versions. Can also be a date, check
  [`available_dates()`](https://docs.ropensci.org/rix/reference/available_dates.md).

- ...:

  Arguments passed on to
  [`rix`](https://docs.ropensci.org/rix/reference/rix.md)

  `system_pkgs`

  :   Vector of characters. List further software you wish to install
      that are not R packages such as command line applications for
      example. You can look for available software on the NixOS website
      <https://search.nixos.org/packages?channel=unstable&from=0&size=50&sort=relevance&type=packages&query=>

  `local_r_pkgs`

  :   Vector of characters, paths to local packages to install. These
      packages need to be in the `.tar.gz` or `.zip` formats and must be
      in the same folder as the generated "default.nix" file.

  `tex_pkgs`

  :   Vector of characters. A set of TeX packages to install. Use this
      if you need to compile `.tex` documents, or build PDF documents
      using Quarto. If you don't know which package to add, start by
      adding "amsmath". See the
      `vignette("d2- installing-system-tools-and-texlive-packages-in-a-nix-environment")`
      for more details.

  `py_conf`

  :   List. A list containing two or three elements: `py_version`,
      `py_pkgs`, and optionally `py_src_dir`. `py_version` should be in
      the form `"3.12"` for Python 3.12, and `py_pkgs` should be an
      atomic vector of package names (e.g.,
      `py_pkgs = c("polars", "plotnine", "great-tables")`). If Python
      packages are requested but `{reticulate}` is not in the list of R
      packages, the user will be warned that they may want to add it.
      When `py_conf` packages are requested, the `RETICULATE_PYTHON`
      environment variable is set to ensure the Nix environment does not
      use a system-wide Python installation. If you are developing a
      Python package, set `"mypackage/src"` or just `"src"`). This adds
      `PYTHONPATH` to the shell hook so your package can be imported
      without installation. This is the Nix equivalent of
      `pip install -e .` (editable install). Note: if `"uv"` is in
      `system_pkgs`, `LD_LIBRARY_PATH` is automatically configured for
      dynamic library loading (required by packages like numpy). You can
      also install packages from git or PyPI by adding `git_pkgs` (list
      of lists with `package_name`, `repo_url`, `commit`) and
      `pypi_pkgs` (vector of package names, optionally with version
      `name@version`) to the `py_conf` list.

  `jl_conf`

  :   List. A list of two elements, `jl_version` and `jl_conf`.
      `jl_version` must be of the form `"1.10"` for Julia 1.10. Leave
      empty or use an empty string to use the latest version, or use
      `"lts"` for the long term support version. `jl_conf` must be an
      atomic vector of packages names, for example
      `jl_conf = c("TidierData", "TidierPlots")`.

  `ide`

  :   Character, defaults to "none". If you wish to use RStudio to work
      interactively use "rstudio" or "rserver" for the server version.
      Use "code" for Visual Studio Code or "codium" for Codium, or
      "positron" for Positron. You can also use "radian", an interactive
      REPL. This will install a project-specific version of the chosen
      editor which will be differrent than the one already present in
      your system (if any). For other editors or if you want to use an
      editor already installed on your system (which will require some
      configuration to make it work seamlessly with Nix shells see the
      [`vignette("configuring-ide")`](https://docs.ropensci.org/rix/articles/configuring-ide.md)
      for configuration examples), use "none". Please be aware that VS
      Code and Positron are not free software. To facilitate their
      installation,
      [`rix()`](https://docs.ropensci.org/rix/reference/rix.md)
      automatically enables a required setting without prompting the
      user for confirmation. See the "Details" section below for more
      information.

  `overwrite`

  :   Logical, defaults to FALSE. If TRUE, overwrite the `default.nix`
      file in the specified path.

  `print`

  :   Logical, defaults to FALSE. If TRUE, print `default.nix` to
      console.

  `message_type`

  :   Character. Message type, defaults to `"simple"`, which gives
      minimal but sufficient feedback. Other values are currently
      `"quiet`, which generates the files without message, and
      `"verbose"`, displays all the messages.

  `shell_hook`

  :   Character of length 1, defaults to `NULL`. Commands added to the
      `shellHook` variable are executed when the Nix shell starts. So by
      default, using `nix-shell default.nix` will start a specific
      program, possibly with flags (separated by space), and/or do shell
      actions. You can for example use `shell_hook = R`, if you want to
      directly enter the declared Nix R session when dropping into the
      Nix shell.

## Value

Nothing, this function is called for its side effects only, unless
`return_rix_call = TRUE` in which case an unevaluated call to
[`rix()`](https://docs.ropensci.org/rix/reference/rix.md) is returned

## Details

In order for this function to work properly, we recommend not running it
inside the same folder as an existing `{renv}` project. Instead, run it
from a new, empty directory which path you pass to `project_path`, and
use `renv_lock_path` to point to the `renv.lock` file in the original
`{renv}` folder. We recommend that you start from an empty folder to
hold your new Nix project, and copy the `renv.lock` file only (not any
of the other files and folders generated by `{renv}`) and then call
`renv2nix()` there. For more details, see
[`vignette("renv2nix")`](https://docs.ropensci.org/rix/articles/renv2nix.md).

## Examples

``` r
if (FALSE) { # \dontrun{
# if the lock file is in another folder
renv2nix(
  renv_lock_path = "path/to/original/renv_project/renv.lock",
  project_path = "path/to/rix_project"
)
# you could also copy the renv.lock file in the folder of the Nix
# project (don’t copy any other files generated by `{renv}`)
renv2nix(
  renv_lock_path = "path/to/rix_project/renv.lock",
  project_path = "path/to/rix_project"
)
} # }
```
