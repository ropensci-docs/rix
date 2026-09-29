# Evaluate Function in R or Shell Command via `nix-shell` Environment

This function needs an installation of Nix. `with_nix()` has two effects
to run code in isolated and reproducible environments.

1.  Evaluate a function in R or a shell command via the `nix-shell`
    environment (Nix expression for custom software libraries; involving
    pinned versions of R and R packages via Nixpkgs)

2.  If no error, return the result object of `expr` in `with_nix()` into
    the current R session.

## Usage

``` r
with_nix(
  expr,
  program = c("R", "shell"),
  project_path = ".",
  message_type = c("simple", "quiet", "verbose")
)
```

## Arguments

- expr:

  Single R function or call, or character vector of length one with
  shell command and possibly options (flags) of the command to be
  invoked. For `program = R`, you can both use a named or an anonymous
  function. The function provided in `expr` should not evaluate when you
  pass arguments, hence you need to wrap your function call like
  `function() your_fun(arg_a = "a", arg_b = "b")`, to avoid evaluation
  and make sure `expr` is a function (see details and examples).

- program:

  String stating where to evaluate the expression. Either `"R"`, the
  default, or `"shell"`. `where = "R"` will evaluate the expression via
  `RScript` and `where = "shell"` will run the system command in
  `nix-shell`.

- project_path:

  Path to the folder where the `default.nix` file resides. The default
  is `"."`, which is the working directory in the current R session.
  This approach also useful when you have different subfolders with
  separate software environments defined in different `default.nix`
  files.

- message_type:

  String how detailed output is. Currently, there is either `"simple"`
  (default), `"quiet` or `"verbose"`, which shows the script that runs
  via `nix-shell`.

## Value

- if `program = "R"`, R object returned by function given in `expr` when
  evaluated via the R environment in `nix-shell` defined by Nix
  expression.

- if `program = "shell"`, list with the following elements:

  - `status`: exit code

  - `stdout`: character vector with standard output

  - `stderr`: character vector with standard error of `expr` command
    sent to a command line interface provided by a Nix package.

## Details

`with_nix()` gives you the power of evaluating a main function `expr`
and its function call stack that are defined in the current R session in
an encapsulated nix-R session defined by Nix expression (`default.nix`),
which is located in at a distinct project path (`project_path`).

`with_nix()` is very convenient because it gives direct code feedback in
read-eval-print-loop style, which gives a direct interface to the very
reproducible infrastructure-as-code approach offered by Nix and Nixpkgs.
You don't need extra efforts such as setting up DevOps tooling like
Docker and domain specific tools like {renv} to control complex software
environments in R and any other language. It is for example useful for
the following purposes.

1.  test compatibility of custom R code and software/package
    dependencies in development and production environments

2.  directly stream outputs (returned objects), messages and errors from
    any command line tool offered in Nixpkgs into an R session.

3.  Test if evolving R packages change their behavior for given
    unchanged R code, and whether they give identical results or not.

`with_nix()` can evaluate both R code from a nix-R session within
another nix-R session, and also from a host R session (i.e., on macOS or
Linux) within a specific nix-R session. This feature is useful for
testing the reproducibility and compatibility of given code across
different software environments. If testing of different sets of
environments is necessary, you can easily do so by providing Nix
expressions in custom `.nix` or `default.nix` files in different
subfolders of the project.

[`rix_init()`](https://docs.ropensci.org/rix/reference/rix_init.md) is
run automatically to generate a custom `.Rprofile` file for the subshell
in `project_dir`. The defaults in that file ensure that only R packages
from the Nix store, that are defined in the subshell `.nix` file are
loaded and system's libraries are excluded.

To do its job, `with_nix()` heavily relies on patterns that manipulate
language expressions (aka computing on the language) offered in base R
as well as the {codetools} package by Luke Tierney.

Some of the key steps that are done behind the scene:

1.  recursively find, classify, and export global objects (globals) in
    the call stack of `expr` as well as propagate R package environments
    found.

2.  Serialize (save to disk) and deserialize (read from disk) dependent
    data structures as `.Rds` with necessary function arguments
    provided, any relevant globals in the call stack, packages, and
    `expr` outputs returned in a temporary directory.

3.  Use pure `nix-shell` environments to execute a R code script
    reconstructed catching expressions with quoting; it is launched by
    commands like this via `{sys}` by Jeroen Ooms:
    `nix-shell --pure --run "Rscript --vanilla"`.

## See also

Other Nix execution:
[`nix_build()`](https://docs.ropensci.org/rix/reference/nix_build.md)

## Examples

``` r
if (FALSE) { # \dontrun{
# create an isolated, runtime-pure R setup via Nix
project_path <- "./sub_shell"
rix_init(
  project_path = project_path,
  rprofile_action = "create_missing"
)
# generate nix environment in `default.nix`
rix(
  r_ver = "4.2.0",
  project_path = project_path
)
# evaluate function in Nix-R environment via `nix-shell` and `Rscript`,
# stream messages, and bring output back to current R session
out <- with_nix(
  expr = function(mtcars) nrow(mtcars),
  program = "R", project_path = project_path,
  message_type = "simple"
)

# There no limit in the complexity of function call stacks that `with_nix()`
# can possibly handle; however, `expr` should not evaluate and
# needs to be a function for `program = "R"`. If you want to pass the
# a function with arguments, you can do like this
get_sample <- function(seed, n) {
  set.seed(seed)
  out <- sample(seq(1, 10), n)
  return(out)
}

out <- with_nix(
  expr = function() get_sample(seed = 1234, n = 5),
  program = "R",
  project_path = ".",
  message_type = "simple"
)

## You can also attach packages with `library()` calls in the current R
## session, which will be exported to the nix-R session.
## Other option: running system commands through `nix-shell` environment.
} # }
```
