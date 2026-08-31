# Run a Targets Pipeline on GitHub Actions

Run a Targets Pipeline on GitHub Actions

## Usage

``` r
tar_nix_ga()
```

## Value

Nothing, copies file to a directory.

## Details

This function puts a `.yaml` file inside the `.github/workflows/`
folders on the root of your project. This workflow file will use the
projects `default.nix` file to generate the development environment on
GitHub Actions and will then run the projects {targets} pipeline. Make
sure to give read and write permissions to the GitHub Actions bot.

## See also

Other CI/CD:
[`ga_cachix()`](https://docs.ropensci.org/rix/reference/ga_cachix.md)

## Examples

``` r
if (FALSE) { # \dontrun{
tar_nix_ga()
} # }
```
