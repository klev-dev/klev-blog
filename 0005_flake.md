[//]: # ({"title": "direnv and flakes", "date": "2023-12-24"})

direnv and (nix) flakes
===============

To ease the development and provide easily reproducible environment, at [klev](https://klev.dev) we use a combination of [direnv](https://direnv.net/), [nix](https://nixos.org/) and [flakes](https://nixos.wiki/wiki/Flakes). With all these, as soon as you enter our main project directory, all relevant tools and dependencies are installed and made available to use by the developer.

direnv
------

According to [direnv main page](https://direnv.net/):

> `direnv` is an extension for your shell. It augments existing shells with a new feature that can load and unload environment variables depending on the current directory.

Said another way, `direnv` allows its users to define a set of environment variables that are active only while they are in a certain directory (or its children). The concrete file we use in [klev](https://klev.dev) looks like this:

```
# .envrc
use flake
dotenv
layout go
```

The first time you enter `direnv` directory, you'll need to run `direnv allow` to grant `direnv` permission to run. Here is an example of interaction, when entering [klev](https://klev.dev)'s directory:

```
$ cd klev
direnv: error ~/klev/.envrc is blocked. Run `direnv allow` to approve its content

$ direnv allow
direnv: loading ~/klev/.envrc
direnv: using flake
direnv: export +AR +AS +CC +CONFIG_SHELL +CXX +DETERMINISTIC_BUILD ...
```

Our `.envrc` at [klev](https://klev.dev) is simple - it does the following:
 * `use flake` - load a build environment using flakes (next section)
 * `dotenv` - load additional environment variables from `.env`
 * `layout go` - set `$GOPATH` to “$(direnv_layout_dir)/go” (e.g. `.direnv/go`) to isolate this project specific go env.

nix and flakes
--------------

According to [nix main page](https://nixos.org/):

> Nix is a tool that takes a unique approach to package management and system configuration. Learn how to make reproducible, declarative and reliable systems. 

While full discussion of what is nix is beyond the scope of this tutorial, the most relevant for our discussion is using it for declarative development environments - it allows us to describe what we use in our development declaratively and it will pull the dependencies out of it store. 

According to [flakes main page](https://nixos.wiki/wiki/Flakes):

> Flakes is a feature of managing Nix packages to simplify usability and improve reproducibility of Nix installations.

In my (probably naive) view, flakes is a way to be even more declarative then pure nix. It takes a `flake.nix` file, which defines both inputs (which in pure nix are pulled from the env) and outputs, e.g. what apps, packages and shells are going to be available after evaluating the `flake.nix` file.

Here is an example of `flake.nix` used by [klev](https://klev.dev):

```
{
  description = "A flake for klev.dev project";

  inputs = {
    nixpkgs.url = "github:nixos/nixpkgs/nixpkgs-unstable";
    flake-utils.url = "github:numtide/flake-utils";
  };

  outputs = { self, nixpkgs, flake-utils }:
    flake-utils.lib.eachDefaultSystem (system:
      let
        pkgs = import nixpkgs { inherit system; config.allowUnfree = true; };
      in
      {
        devShell = pkgs.mkShell {
          LC_ALL="C.UTF-8";
          buildInputs = with pkgs; [
            go
            caddy
            terraform
            ansible
          ];
        };
      });
}
```

Our `flake.nix` starts with `description`. Then we define the `inputs`, e.g. what things we'll use later on:

```
...
  inputs = {
    nixpkgs.url = "github:nixos/nixpkgs/nixpkgs-unstable";
    flake-utils.url = "github:numtide/flake-utils";
  };
...
```

This specifies that we we'll be using the `unstable` channel for nix packages (as opposed to a released one, such as `23.11`). We'll also be using `flake-utils`, which is a collection of utilities to make using flakes easier.

Then we define the `output`, e.g. what will be available to us:

```
...
  outputs = { self, nixpkgs, flake-utils }:
    flake-utils.lib.eachDefaultSystem (system:
      let
        pkgs = import nixpkgs { inherit system; config.allowUnfree = true; };
      in
      {
...
```
Here we define two interesting things: 
 * `flake-utils.lib.eachDefaultSystem` allows us to have the same set of outputs for each nix supported system
 * `inherit system; config.allowUnfree = true;` allow us to configure using non-free software (in our case `terraform`)

Finally, we describe what our shell will look like:

```
...
        devShell = pkgs.mkShell {
          LC_ALL="C.UTF-8";
          buildInputs = with pkgs; [
            go
            caddy
            terraform
            ansible
          ];
        };
...
```

This part describe we'll be using the following packages:
 * `go` - which is the official golang package
 * `caddy` - the [caddy http server](https://caddyserver.com/)
 * `terraform` - [terraform](https://www.terraform.io/) to manage our infra
 * `ansible` - [ansible](https://www.ansible.com/) to deploy

> **NOTE** `LC_ALL="C.UTF-8";` is required to run ansible on some of our systems.

Here how it looks for [klev](https://klev.dev):

```
$ ansible --version
ansible may be found in the following packages:
  extra/ansible-core 2.15.5-1	/usr/bin/ansible

$ cd klev
direnv: loading ~/Sources/klev/.envrc
direnv: using flake
direnv: export +AR +AS +CC +CONFIG_SHELL +CXX ...

$ ansible --version
ansible [core 2.15.5]
...
```

Conclusion
----------

By using direnv and flakes, we create a customized development environment once developers enter the relevant project dir. This environment is declarative and the same for each developer, as well as across different systems. We only need to install direnv and nix to empower working on all kinds of projects, each with its own dependencies and tools.

&#128075; Enjoy hacking!
