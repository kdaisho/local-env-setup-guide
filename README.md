# Local environment setup guide

## Nix and direnv

For managing dependencies, we use Nix and direnv. Using these two tools together makes managing them
significantly easier.

### [`Nix`](https://nixos.org/)

A purely functional package manager.

### [direnv](https://direnv.net/)

It can load and unload environment variables depending on the current directory, allowing developers to maintain different dependencies per directory. For this, we recommend
installing [nix-direnv](https://github.com/nix-community/nix-direnv) (a variant maintained by nix
community).

## Installation

### Nix

```
sh <(curl -L https://nixos.org/nix/install)
```

https://nixos.org/download.html#nix-install-macos

### nix-direnv

There are a few ways to install it. Installing via
[home-manager](https://nix-community.github.io/home-manager) is recommended.

#### Installing home-manager

Follow _1.1. Standalone installation_ from the link below.

https://nix-community.github.io/home-manager/index.html#sec-install-standalone

Follow _2.1. Configuration Example_ from the link below.

https://nix-community.github.io/home-manager/index.html#ch-usage

After successful installation, you should be able to print the version.

```
home-manager --version
```

#### Installing nix-direnv via home-manager

In `$HOME/.config/nixpkgs/home.nix` add

```
{ pkgs, ... }:

{
  # ...other config, other config...

  programs.direnv.enable = true;
  programs.direnv.nix-direnv.enable = true;
  # optional for nix flakes support in home-manager 21.11, not required in home-manager unstable or 22.05
  programs.direnv.nix-direnv.enableFlakes = true;

  programs.bash.enable = true;
  # OR
  programs.zsh.enable = true;
  # Or any other shell you're using.
}
```

To protect your nix-shell against garbage collection you also need to add these options to your Nix
configuration (`/etc/nix/nix.conf`).

```
keep-derivations = true
keep-outputs = true
```

### Test

Restart your shell, then run

```
nix --version
```

```
direnv --version
```

Both command should print the version.

## Demo

For demonstration, we're installing `deno` and `docker-compose` while setting scope within `test` and its subdirectories.

Create a directory `test` then navigate into it.

```bash
mkdir test && cd test
```

Create a file `shell.nix` then add

```
let
  pkgsUnstable = import (fetchTarball "http://nixos.org/channels/nixos-unstable/nixexprs.tar.xz") {};
in
  pkgsUnstable.mkShell {
    buildInputs = [
      pkgsUnstable.deno
      pkgsUnstable.docker-compose
    ];
  }
```

Create a file `.envrc` then add

```
use nix
```

Once you exit the editor, direnv prompts you to run `direnv allow` for approval. Run below to apply
the change and install dependencies listed in shell.nix.

```
direnv allow .
```

Installing all dependencies might take some time, but it shouldn't be more than a few minutes.
If you encounter this, abort the process (CTRL-C), then try different version of dependencies. [Nix Packages](https://search.nixos.org/packages)

### Test

Both command below should print the version.

```
deno --version
```

```
docker-compose --version
```

Once navigating to the parent directory, you should lose access to the dependencies.

```bash
cd ../ && deno --version
# zsh: command not found: deno
```
