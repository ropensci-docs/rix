# Package index

## Core functions

The main functions for generating and initializing Nix environments.

- [`rix()`](https://docs.ropensci.org/rix/reference/rix.md) : Generate a
  Nix Expression That Builds a Reproducible Development Environment
- [`rix_init()`](https://docs.ropensci.org/rix/reference/rix_init.md) :
  Initiate and Maintain an Isolated, Project-Specific, and Runtime-Pure
  R Setup via Nix.

## Nix execution

Functions to build Nix expressions and run code inside Nix environments.

- [`nix_build()`](https://docs.ropensci.org/rix/reference/nix_build.md)
  :

  Invoke Shell Command `nix-build` from an R Session

- [`with_nix()`](https://docs.ropensci.org/rix/reference/with_nix.md) :

  Evaluate Function in R or Shell Command via `nix-shell` Environment

## Available versions

Functions to query available R versions and dates for environment
generation.

- [`available_dates()`](https://docs.ropensci.org/rix/reference/available_dates.md)
  : List Available Dates for R and Bioconductor Releases
- [`available_df()`](https://docs.ropensci.org/rix/reference/available_df.md)
  : Return Data Frame with R, Bioc Versions and Supported Platforms
- [`available_r()`](https://docs.ropensci.org/rix/reference/available_r.md)
  : List Available R Versions from the rstats-on-nix Fork of Nixpkgs

## CI/CD integration

Functions for integrating rix with GitHub Actions and Cachix.

- [`ga_cachix()`](https://docs.ropensci.org/rix/reference/ga_cachix.md)
  : Build an Environment on GitHub Actions and Cache It on Cachix
- [`tar_nix_ga()`](https://docs.ropensci.org/rix/reference/tar_nix_ga.md)
  : Run a Targets Pipeline on GitHub Actions

## Project setup

Functions for converting renv projects, setting up caches, and creating
launchers.

- [`renv2nix()`](https://docs.ropensci.org/rix/reference/renv2nix.md) :

  Convert an `renv.lock` File to a Nix Expression

- [`setup_cachix()`](https://docs.ropensci.org/rix/reference/setup_cachix.md)
  : Configure the rstats-on-nix Binary Cache

- [`make_launcher()`](https://docs.ropensci.org/rix/reference/make_launcher.md)
  : Create a Startup Script to Launch an Editor Inside a Nix Shell
