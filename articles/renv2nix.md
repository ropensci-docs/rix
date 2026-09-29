# Converting renv Projects to Nix Projects

*renv2nix() is early development stages and its arguments may change in
the future*

## Introduction

[renv](https://rstudio.github.io/renv/) is very likely the most popular
package for reproducibility in R.
[renv](https://rstudio.github.io/renv/) generates and manages so-called
`renv.lock` files at the level of a project, and using these files it is
thus possible to have different versions of the same packages on the
same system, without any interference. However,
[renv](https://rstudio.github.io/renv/) doesn’t snapshot R itself, so
different projects with different libraries of packages will end up
using the same version of R, which could lead to issues, especially if
you’re trying to restore an old project which relied on an old version
of R. Also, in some cases, it might be impossible to restore a project
due to incompatibilities at the level of the system-level dependencies,
as these are not managed by [renv](https://rstudio.github.io/renv/). In
our experience, the older the project, the less likely restoring it
using [renv](https://rstudio.github.io/renv/) will succeed.

As explained extensively in this documentation already, Nix handles all
the different pieces of the reproducibility puzzle: versions of R
packages, versions of R, and versions of system-level dependencies are
all handled by Nix. In other words, it’s possible to have per project
complete development environments that are completely reproducible.

While Nix has been around for 20 years, it was never widely adopted by R
users, who instead have already invested a lot of effort into
[renv](https://rstudio.github.io/renv/) and Docker. The goal of
[rix](https://docs.ropensci.org/rix/) is to ease adoption of Nix for R
users for new projects, but there is also a need to convert historical
[renv](https://rstudio.github.io/renv/) projects into Nix.

This is where
[`renv2nix()`](https://docs.ropensci.org/rix/reference/renv2nix.md) can
help.

## Converting an historical renv project

There are two ways that you can convert an historical project: the
recommended way is to copy the historical `renv.lock` file into a new,
empty folder. Do not copy any of the other generated
[renv](https://rstudio.github.io/renv/) files nor folders. From there,
call [`renv2nix()`](https://docs.ropensci.org/rix/reference/renv2nix.md)
like so:

``` r

renv2nix(
  renv_lock_path = "path/to/rix_project/renv.lock",
  project_path = "path/to/rix_project"
)
```

This will generate a `default.nix` and `.Rprofile` files for your Nix
project.

The other way to achieve this is to point the `renv_lock_path` argument
to the historical `renv.lock` file without copying it to a new folder:

``` r

renv2nix(
  renv_lock_path = "path/to/original/renv_project/renv.lock",
  project_path = "path/to/rix_project"
)
```

But this is not the recommended approach. Instead keep the original
`renv.lock` file and the generated `default.nix` files together (ideally
with the call to
[`renv2nix()`](https://docs.ropensci.org/rix/reference/renv2nix.md) in
an R script) and commit everything into version control.

## Starting a new project

If you start a new project, you don’t need to use
[`renv2nix()`](https://docs.ropensci.org/rix/reference/renv2nix.md), as
you could directly use
[`rix()`](https://docs.ropensci.org/rix/reference/rix.md) to generate a
`default.nix` file. However, you could start with generating an
`renv.lock` file using `renv::snapshot()` and then call
[`renv2nix()`](https://docs.ropensci.org/rix/reference/renv2nix.md), to
generate the appropriate `default.nix` file. Because `renv::snapshot()`
does not call `renv::init()` this will not generate any other files that
could interfere with [rix](https://docs.ropensci.org/rix/). Doing this
could be useful if you have the habit of starting writing code and then
would like to generate a `default.nix` file later. We don’t recommend
working like this and instead urge you to start from an environment and
work from the very beginning from that environment.

## Caveats

### Package versions are not exactly the same between the renv.lock and default.nix files

[`renv2nix()`](https://docs.ropensci.org/rix/reference/renv2nix.md) has
two methods, `"fast"` and `"accurate"`. `"fast"` simply lists the
version of R and R packages, and generates a `default.nix` expression
without trying to match package versions exactly. We believe that this
is acceptable for most use cases. The planned `"accurate"` method will
instead try to do its best to match exact package versions. However, due
to how differently Nix and [renv](https://rstudio.github.io/renv/)
handle snapshotting, it will not be possible to match package versions
exactly even with the planned `"accurate"` method. If you absolutely
need a very specific version of a package, there are other ways to
achieve this, by providing for example the required version to the
`r_pkgs` argument of
[`rix()`](https://docs.ropensci.org/rix/reference/rix.md), like this:
`rix(r_pkgs = "dplyr@0.8.0")`. This however tries to build the pacage
from source, which can fail. If this is the case, don’t hesitate to open
an issue. Better handling of individual package versions is planned for
a future release.

### Don’t use the same folder for your Nix and {renv} projects

As stated previously, don’t convert an `renv.lock` file into a
`default.nix` file from a folder that contains the `renv.lock` file and
the [renv](https://rstudio.github.io/renv/) generated `.Rprofile` file
and `renv/` folder. Instead, work from a new empty folder in which you
copy the `renv.lock` file.

### Mind the R version

Many R users do not update R very often, so when they generate an
`renv.lock` file, the `renv.lock` will list an old version of R, but
potentially very recent packages. The way
[rix](https://docs.ropensci.org/rix/) generates `default.nix` files is
by looking at the version of R and then install packages that were
current at that time. So if the `renv.lock` file lists an old version of
R, the packages that will be included in the `default.nix` file will
also be old. You could even end up in a situation where a package is not
available because it only recently got released on CRAN. To avoid
problems, use the `override_r_ver` argument of
[`renv2nix()`](https://docs.ropensci.org/rix/reference/renv2nix.md) to
provide a more recent version of R, that matches roughly when the
packages were released.
