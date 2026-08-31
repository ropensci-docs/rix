# Build an Environment on GitHub Actions and Cache It on Cachix

Build an Environment on GitHub Actions and Cache It on Cachix

## Usage

``` r
ga_cachix(cache_name, path_default)
```

## Arguments

- cache_name:

  String, name of your cache.

- path_default:

  String, relative path (from the root directory of your project) to the
  `default.nix` to build.

## Value

Nothing, copies file to a directory.

## Details

This function puts a `.yaml` file inside the `.github/workflows/`
folders on the root of your project. This workflow file will use the
projects `default.nix` file to generate the development environment on
GitHub Actions and will then cache the created binaries in Cachix.
Create a free account on Cachix to use this action. Refer to
[`vignette("binary-cache")`](https://docs.ropensci.org/rix/articles/binary-cache.md)
for detailed instructions. Make sure to give read and write permissions
to the GitHub Actions bot.

## See also

Other CI/CD:
[`tar_nix_ga()`](https://docs.ropensci.org/rix/reference/tar_nix_ga.md)

## Examples

``` r
if (FALSE) { # \dontrun{
ga_cachix("my-cachix", path_default = "default.nix")
} # }
```
