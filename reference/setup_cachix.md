# Configure the rstats-on-nix Binary Cache

Configure the rstats-on-nix Binary Cache

## Usage

``` r
setup_cachix(nix_conf_path = "~/.config/nix")
```

## Arguments

- nix_conf_path:

  Character, path to folder containing 'nix.conf' file. Defaults to
  `"~/.config/nix"`.

## Value

Nothing; changes a file in the user's home directory.

## Details

This function edits `~/.config/nix/nix.conf` to add the `rstats-on-nix`
public cache as a substituter. The `rstats-on-nix` public cache, hosted
on Cachix, contains many prebuild binaries of R and R packages for
x86_64 Linux and macOS (Intel architectures for packages released before
2021 and Apple Silicon from 2021 onwards). This function automatically
performs a backup of `~/.config/nix/nix.conf`, or creates one if there
is no `nix.conf` file.

This is the recommended approach for configuring the cache, as it works
with both standard Nix installations and Determinate Nix installations.
After running this function, you also need to add yourself to
`trusted-users` so Nix allows you to use the cache. Run one of:

- **Linux**:
  `echo "trusted-users = root $USER" | sudo tee -a /etc/nix/nix.custom.conf && sudo systemctl restart nix-daemon`

- **macOS**:
  `echo "trusted-users = root $USER" | sudo tee -a /etc/nix/nix.custom.conf && sudo launchctl kickstart -k system/org.nixos.nix-daemon`

If you see warnings like "ignoring untrusted substituter", this means
the trusted-users configuration is not in place.

**NixOS users**: This function does not work on NixOS because the Nix
configuration is managed declaratively. Instead, configure the cache in
your system configuration. Without Home Manager, add this to your
`configuration.nix`:

    nix.settings = {
      substituters = [
        "https://cache.nixos.org"
        "https://rstats-on-nix.cachix.org"
      ];
      trusted-public-keys = [
        "cache.nixos.org-1:6NCHdD59X431o0gWypbMrAURkbJ16ZPMQFGspcDShjY="
        "rstats-on-nix.cachix.org-1:vdiiVgocg6WeJrODIqdprZRUrhi1JzhBnXv7aWI6+F0="
      ];
    };

With Home Manager, add this to your home configuration:

    nix.settings = {
      substituters = [
        "https://rstats-on-nix.cachix.org"
      ];
      trusted-public-keys = [
        "rstats-on-nix.cachix.org-1:vdiiVgocg6WeJrODIqdprZRUrhi1JzhBnXv7aWI6+F0="
      ];
    };

Other use cases include: if you somehow mess up `~/.config/nix/nix.conf`
and need to generate a new one from scratch, or if you're using Nix
inside Docker, write a `RUN Rscript -e 'rix::setup_cachix()'` statement
to configure the cache there. Because Docker runs using `root` by
default no need to install the `cachix` client to configure the cache,
running `setup_cachix()` is enough. See the 'z - Advanced topic: Using
Nix inside Docker' vignette for more details.

## Examples

``` r
if (FALSE) { # \dontrun{
setup_cachix()
} # }
```
