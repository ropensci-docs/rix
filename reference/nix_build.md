# Invoke Shell Command `nix-build` from an R Session

Invoke Shell Command `nix-build` from an R Session

## Usage

``` r
nix_build(
  project_path = getwd(),
  message_type = c("simple", "quiet", "verbose"),
  args = NULL
)
```

## Arguments

- project_path:

  Path to the folder where the `default.nix` file resides.

- message_type:

  Character vector with messaging type. Either `"simple"` (default),
  `"quiet"` for no messaging, or `"verbose"`.

- args:

  A character vector of additional arguments to be passed directly to
  the `nix-build` command. If the project directory (i.e.
  `project_path`) is not included in `args`, it will be appended
  automatically.

## Value

Integer of the process ID (PID) of the `nix-build` shell command
launched, if the `nix_build()` call is assigned to an R object.
Otherwise, it will be returned invisibly.

## Details

This function is a wrapper for the `nix-build` command-line interface.
Users can supply any flags supported by `nix-build` via the `args`
parameter. If no custom arguments are provided, only the project
directory is passed.

## See also

Other Nix execution:
[`with_nix()`](https://docs.ropensci.org/rix/reference/with_nix.md)

## Examples

``` r
if (FALSE) { # \dontrun{
  # Run nix-build with default arguments (project directory)
  nix_build()

  # Run nix-build with custom arguments
  nix_build(args = c("--max-jobs", "2", "--quiet"))
} # }
```
