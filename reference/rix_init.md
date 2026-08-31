# Initiate and Maintain an Isolated, Project-Specific, and Runtime-Pure R Setup via Nix.

Creates an isolated project folder for a Nix-R configuration.
`rix::rix_init()` also adds, appends, or updates with or without backup
a custom `.Rprofile` file with code that initializes a startup R
environment without system's user libraries within a Nix software
environment. Instead, it restricts search paths to load R packages
exclusively from the Nix store. Additionally, it makes Nix utilities
like `nix-shell` available to run system commands from the system's
RStudio R session, for both Linux and macOS.

## Usage

``` r
rix_init(
  project_path,
  rprofile_action = c("create_missing", "create_backup", "overwrite", "append"),
  message_type = c("simple", "quiet", "verbose")
)
```

## Arguments

- project_path:

  Character with the folder path to the isolated nix-R project. If the
  folder does not exist yet, it will be created.

- rprofile_action:

  Character. Action to take with `.Rprofile` file destined for
  `project_path` folder. Possible values include `"create_missing"`,
  which only writes `.Rprofile` if it does not yet exist (otherwise does
  nothing) - this is the action set when using
  [`rix()`](https://docs.ropensci.org/rix/reference/rix.md) - ;
  `"create_backup"`, which copies the existing `.Rprofile` to a new
  backup file, generating names with POSIXct-derived strings that
  include the time zone information. A new `.Rprofile` file will be
  written with default code from `rix::rix_init()`; `"overwrite"`
  overwrites the `.Rprofile` file if it does exist; `"append"` appends
  the existing file with code that is tailored to an isolated Nix-R
  project setup.

- message_type:

  Character. Message type, defaults to `"simple"`, which gives minimal
  but sufficient feedback. Other values are currently `"quiet`, which
  writes `.Rprofile` without message, and `"verbose"`, which displays
  the mechanisms implemented to achieve fully controlled R project
  environments in Nix.

## Value

Nothing, this function only has the side-effect of writing a file called
".Rprofile" to the specified path.

## Details

**Enhancement of computational reproducibility for Nix-R environments:**

The primary goal of `rix::rix_init()` is to enhance the computational
reproducibility of Nix-R environments during runtime. Concretely, if you
already have a system or user library of R packages (if you have R
installed through the usual means for your operating system), using
`rix::rix_init()` will prevent Nix-R environments to load packages from
the user library which would cause issues. Notably, no restart is
required as environmental variables are set in the current session, in
addition to writing an `.Rprofile` file. This is particularly useful to
make [`with_nix()`](https://docs.ropensci.org/rix/reference/with_nix.md)
evaluate custom R functions from any "Nix-to-Nix" or "System-to-Nix" R
setups. It introduces two side-effects that take effect both in a
current or later R session setup:

1.  **Adjusting `R_LIBS_USER` path:** By default, the first path of
    `R_LIBS_USER` points to the user library outside the Nix store (see
    also [`base::.libPaths()`](https://rdrr.io/r/base/libPaths.html)).
    This creates friction and potential impurity as R packages from the
    system's R user library are loaded. While this feature can be useful
    for interactively testing an R package in a Nix environment before
    adding it to a `.nix` configuration, it can have undesired effects
    if not managed carefully. A major drawback is that all R packages in
    the `R_LIBS_USER` location need to be cleaned to avoid loading
    packages outside the Nix configuration. Issues, especially on macOS,
    may arise due to segmentation faults or incompatible linked system
    libraries. These problems can also occur if one of the (reverse)
    dependencies of an R package is loaded along the process.

2.  **Make Nix commands available when running system commands from
    RStudio:** In a host RStudio session not launched via Nix
    (`nix-shell`), the environmental variables from `~/.zshrc` or
    `~/.bashrc` may not be inherited. Consequently, Nix command line
    interfaces like `nix-shell` might not be found. The `.Rprofile` code
    written by `rix::rix_init()` ensures that Nix command line programs
    are accessible by adding the path of the "bin" directory of the
    default Nix profile, `"/nix/var/nix/profiles/default/bin"`, to the
    `PATH` variable in an RStudio R session.

These side effects are particularly recommended when working in flexible
R environments, especially for users who want to maintain both the
system's native R setup and utilize Nix expressions for reproducible
development environments. This init configuration is considered pivotal
to enhance the adoption of Nix in the R community, particularly until
RStudio in Nixpkgs is packaged for macOS. We recommend calling
`rix::rix_init()` prior to comparing R code ran between two software
environments with
[`rix::with_nix()`](https://docs.ropensci.org/rix/reference/with_nix.md).

`rix::rix_init()` is called automatically by
[`rix::rix()`](https://docs.ropensci.org/rix/reference/rix.md) when
generating a `default.nix` file, and when called by
[`rix::rix()`](https://docs.ropensci.org/rix/reference/rix.md) will only
add the `.Rprofile` if none exists. In case you have a custom
`.Rprofile` that you wish to keep using, but also want to benefit from
what `rix_init()` offers, manually call it and set the `rprofile_action`
to `"append"`.

## See also

[`with_nix()`](https://docs.ropensci.org/rix/reference/with_nix.md)

Other core functions:
[`rix()`](https://docs.ropensci.org/rix/reference/rix.md)

## Examples

``` r
if (FALSE) { # \dontrun{
# create an isolated, runtime-pure R setup via Nix
project_path <- "./sub_shell"
if (!dir.exists(project_path)) dir.create(project_path)
rix_init(
  project_path = project_path,
  rprofile_action = "create_missing",
  message_type = c("simple")
)
} # }
```
