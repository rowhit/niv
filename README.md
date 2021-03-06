# niv

![GitHub Actions](https://github.com/nmattia/niv/workflows/Test/badge.svg?branch=master)
[![Netlify Status](https://api.netlify.com/api/v1/badges/48532eaa-259f-4ca2-aadf-67f7c6b957fd/deploy-status)](https://niv.nmattia.com)

Painless dependencies for [Nix] projects. Read more in the [Getting started](#getting-started) section below.

<p align="center">
    <img src="https://niv.nmattia.com/niv.svg">
</p>


* [Install](#install)
* [Build](#build)
* [Usage](#usage)
* [FAQ](#frequently-asked-questions)

## Install

`niv` is available in [`nixpkgs`](https://github.com/NixOS/nixpkgs) as `niv`:

``` bash
$ nix-env -iA nixpkgs.niv
```

Alternatively, run the following command to install the development version:

``` bash
$ nix-env -iA niv -f https://github.com/nmattia/niv/tarball/master \
    --substituters https://niv.cachix.org \
    --trusted-public-keys niv.cachix.org-1:X32PCg2e/zAm3/uD1ScqW2z/K0LtDyNV7RdaxIuLgQM=
```

## Build

Inside the provided nix shell:

``` bash
$ repl
```

Run the test suite with this command:

``` bash
$ ./script/test
```

## Usage

`niv` simplifies [adding](#add) and [updating](#update) dependencies in Nix
projects. It uses a single file, `nix/sources.json`, where it stores the data
necessary for fetching and updating the packages.

* [Add](#add): inserts a package in `nix/sources.json`.
* [Update](#update): updates one or all packages in `nix/sources.json`.
* [Drop](#drop): deletes a package from `nix/sources.json`.

`niv` has some utility functions:

* [Init](#init): bootstraps a Nix project, in particular creates a
  `nix/sources.json` file containing `niv` and `nixpkgs` as well as a
  `nix/sources.nix` file that returns the sources as a Nix object.
* [Show](#show): shows the packages' information.
* [Modify](#modify): modifies attributes _without_ performing an update.

### Configuration

The following environment variables are read by `niv`:

| Name            | Note |
| --------------- | ---- |
| GITHUB_TOKEN    | When set, the value is used to authenticate GitHub API requests. |
| GITHUB_HOST     | The GitHub host to use when fetching packages. Port may be appended here. |
| GITHUB_API_HOST | The host used when performing GitHub API requests. Use `GITHUB_API_PORT` for specifying the port. |
| GITHUB_API_PORT | The port used when performing GitHub API requests. Defaults to `443` for secure requests. Defaults to `80` for insecure requests. See also: `GITHUB_INSECURE`. |
| GITHUB_INSECURE | When set to anything but the empty string, requests are performed over `http` instead of `https`. |
| GITHUB_PATH     | The base path used when performing GitHub API requests. |

The next two sections cover [common use cases](#getting-started) and [full command
description](#commands).

### Getting started

This section covers common use cases:

* [Bootstrapping a Nix project](#bootstrapping-a-nix-project).
* [Tracking a different nixpkgs branch](#tracking-a-nixpkgs-branch).
* [Importing packages from GitHub](#importing-packages-from-github).
* [Fetching packages from custom URLs](#using-custom-urls).

#### Bootstrapping a Nix project

Use the `init` command when starting a new Nix project or when porting an
existing Nix project to niv:

``` shell
$ niv init
...
$ tree
.
└── nix
    ├── sources.json
    └── sources.nix

1 directory, 2 files
```

The file `nix/sources.json` is the file used by niv to store versions and is
initialized with niv and nixpkgs:

``` json
{
    "nixpkgs": {
        "url": "https://github.com/NixOS/nixpkgs/archive/109a28ab954a0ad129f7621d468f829981b8b96c.tar.gz",
        "owner": "NixOS",
        "branch": "nixos-19.09",
        "url_template": "https://github.com/<owner>/<repo>/archive/<rev>.tar.gz",
        "repo": "nixpkgs",
        "sha256": "12wnxla7ld4cgpdndaipdh3j4zdalifk287ihxhnmrzrghjahs3q",
        "description": "Nix Packages collection",
        "rev": "109a28ab954a0ad129f7621d468f829981b8b96c"
    },
    "niv": {
        "homepage": "https://github.com/nmattia/niv",
        "url": "https://github.com/nmattia/niv/archive/72e77204544527279e8f1e2d982d29503482b8f4.tar.gz",
        "owner": "nmattia",
        "branch": "master",
        "url_template": "https://github.com/<owner>/<repo>/archive/<rev>.tar.gz",
        "repo": "niv",
        "sha256": "1zjcyzxhq9iwxh58j5d7sx1vz5s3r1f6gpmnfgj2a3rxmclwvn3c",
        "description": "Easy dependency management for Nix projects",
        "rev": "72e77204544527279e8f1e2d982d29503482b8f4"
    }
}
```

To use those dependencies `import` the file `nix/sources.nix`, e.g.:

``` nix
{ sources ? import ./sources.nix }:     # import the sources
with
  { overlay = _: pkgs:
      { niv = import sources.niv {};    # use the sources :)
      };
  };
import sources.nixpkgs                  # and use them again!
  { overlays = [ overlay ] ; config = {}; }
```

#### Tracking a nixpkgs branch

The `init` command sets the `nix/sources.json` file to track the latest commit
present on nixpkgs 19.09 when the command was run. Run the following command to
update it:

``` shell
$ niv update nixpkgs
```

To change the branch being tracked run this command:

``` shell
$ niv update nixpkgs -b nixos-19.09     # equivalent to --branch nixos-19.09
```

#### Importing packages from GitHub

The `add` command will infer information about the package being added, when
possible. This works very well for GitHub repositories. Run this command to add
[jq] to your project:


``` shell
$ niv add stedolan/jq
```

The following data was added in `nix/sources.json` for `jq`:

``` json
{
  "homepage": "http://stedolan.github.io/jq/",
  "url": "https://github.com/stedolan/jq/archive/9fa2e51099c55af56e3e541dc4b399f11de74abe.tar.gz",
  "owner": "stedolan",
  "branch": "master",
  "url_template": "https://github.com/<owner>/<repo>/archive/<rev>.tar.gz",
  "repo": "jq",
  "sha256": "0819rvk8057qgcqvgn7fpldvly2pfdw9fxcjrlqa8gr59p8a1cic",
  "description": "Command-line JSON processor",
  "rev": "9fa2e51099c55af56e3e541dc4b399f11de74abe"
}
```

#### Using custom URLs

It is possible to use niv to fetch packages from custom URLs. Run this command
to add the Haskell compiler [GHC] to your `nix/sources.json`:

``` shell
$ niv add ghc   \
    -v 8.4.3    \
    -t 'https://downloads.haskell.org/~ghc/<version>/ghc-<version>-i386-deb8-linux.tar.xz'
```

The option `-v` sets the "version" attribute to `8.4.3`. The option `-t` sets a
template that can be reused by niv when fetching a new URL (see the
documentation for [add](#add) and [update](#update)).

For updating the version of GHC used run this command:

``` shell
$ niv update ghc -v 8.6.2
```

### Commands

```
niv - dependency manager for Nix projects

version: 0.2.16

Usage: niv [-s|--sources-file FILE] COMMAND

Available options:
  -s,--sources-file FILE   Use FILE instead of nix/sources.json
  -h,--help                Show this help text

Available commands:
  init                     Initialize a Nix project. Existing files won't be
                           modified.
  add                      Add a GitHub dependency
  show                     
  update                   Update dependencies
  modify                   Modify dependency attributes without performing an
                           update
  drop                     Drop dependency
```

#### Add

```
Examples:

  niv add stedolan/jq
  niv add NixOS/nixpkgs -n nixpkgs -b nixpkgs-unstable
  niv add my-package -v alpha-0.1 -t http://example.com/archive/<version>.zip

Usage: niv add PACKAGE [-n|--name NAME] ([-a|--attribute KEY=VAL] |
               [-s|--string-attribute KEY=VAL] | [-b|--branch BRANCH] |
               [-o|--owner OWNER] | [-r|--rev REV] | [-v|--version VERSION] |
               [-t|--template URL] | [-T|--type TYPE])
  Add a GitHub dependency

Available options:
  -n,--name NAME           Set the package name to <NAME>
  -a,--attribute KEY=VAL   Set the package spec attribute <KEY> to <VAL>, where
                           <VAL> may be JSON.
  -s,--string-attribute KEY=VAL
                           Set the package spec attribute <KEY> to <VAL>.
  -b,--branch BRANCH       Equivalent to --attribute branch=<BRANCH>
  -o,--owner OWNER         Equivalent to --attribute owner=<OWNER>
  -r,--rev REV             Equivalent to --attribute rev=<REV>
  -v,--version VERSION     Equivalent to --attribute version=<VERSION>
  -t,--template URL        Used during 'update' when building URL. Occurrences
                           of <foo> are replaced with attribute 'foo'.
  -T,--type TYPE           The type of the URL target. The value can be either
                           'file' or 'tarball'. If not set, the value is
                           inferred from the suffix of the URL.
  -h,--help                Show this help text

Experimental commands:
  git                      Add a git dependency. Experimental.
  github                   Add a GitHub dependency
  local                    Add a local dependency. Experimental.
```

#### Update

```
Examples:
  
  niv update                     # update all packages
  niv update nixpkgs             # update nixpkgs
  niv update my-package -v beta-0.2 # update my-package to version "beta-0.2"

Usage: niv update [PACKAGE] ([-a|--attribute KEY=VAL] |
                  [-s|--string-attribute KEY=VAL] | [-b|--branch BRANCH] |
                  [-o|--owner OWNER] | [-r|--rev REV] | [-v|--version VERSION] |
                  [-t|--template URL] | [-T|--type TYPE])
  Update dependencies

Available options:
  -a,--attribute KEY=VAL   Set the package spec attribute <KEY> to <VAL>, where
                           <VAL> may be JSON.
  -s,--string-attribute KEY=VAL
                           Set the package spec attribute <KEY> to <VAL>.
  -b,--branch BRANCH       Equivalent to --attribute branch=<BRANCH>
  -o,--owner OWNER         Equivalent to --attribute owner=<OWNER>
  -r,--rev REV             Equivalent to --attribute rev=<REV>
  -v,--version VERSION     Equivalent to --attribute version=<VERSION>
  -t,--template URL        Used during 'update' when building URL. Occurrences
                           of <foo> are replaced with attribute 'foo'.
  -T,--type TYPE           The type of the URL target. The value can be either
                           'file' or 'tarball'. If not set, the value is
                           inferred from the suffix of the URL.
  -h,--help                Show this help text
```

#### Modify

```
Examples:

  niv modify nixpkgs -v beta-0.2
  niv modify nixpkgs -a branch=nixpkgs-unstable

Usage: niv modify PACKAGE [-n|--name NAME] ([-a|--attribute KEY=VAL] |
                  [-s|--string-attribute KEY=VAL] | [-b|--branch BRANCH] |
                  [-o|--owner OWNER] | [-r|--rev REV] | [-v|--version VERSION] |
                  [-t|--template URL] | [-T|--type TYPE])
  Modify dependency attributes without performing an update

Available options:
  -n,--name NAME           Set the package name to <NAME>
  -a,--attribute KEY=VAL   Set the package spec attribute <KEY> to <VAL>, where
                           <VAL> may be JSON.
  -s,--string-attribute KEY=VAL
                           Set the package spec attribute <KEY> to <VAL>.
  -b,--branch BRANCH       Equivalent to --attribute branch=<BRANCH>
  -o,--owner OWNER         Equivalent to --attribute owner=<OWNER>
  -r,--rev REV             Equivalent to --attribute rev=<REV>
  -v,--version VERSION     Equivalent to --attribute version=<VERSION>
  -t,--template URL        Used during 'update' when building URL. Occurrences
                           of <foo> are replaced with attribute 'foo'.
  -T,--type TYPE           The type of the URL target. The value can be either
                           'file' or 'tarball'. If not set, the value is
                           inferred from the suffix of the URL.
  -h,--help                Show this help text
```

#### Drop

```
Examples:

  niv drop jq
  niv drop my-package version

Usage: niv drop PACKAGE [ATTRIBUTE]
  Drop dependency

Available options:
  -h,--help                Show this help text
```

#### Init

```
Usage: niv init ([--no-nixpkgs] | [-b|--nixpkgs-branch ARG]
                [--nixpkgs OWNER/REPO])
  Initialize a Nix project. Existing files won't be modified.

Available options:
  --no-nixpkgs             Don't add a nixpkgs entry to sources.json.
  -b,--nixpkgs-branch ARG  The nixpkgs branch to use. (default: "release-20.03")
  --nixpkgs OWNER/REPO     Use a custom nixpkgs repository from
                           GitHub. (default: NixOS/nixpkgs)
  -h,--help                Show this help text
```

#### show

```
Usage: niv show [PACKAGE]

Available options:
  -h,--help                Show this help text
```

[Nix]: https://nixos.org/nix/
[jq]: https://stedolan.github.io/jq/
[GHC]: https://www.haskell.org/ghc/


## Frequently Asked Questions

* [Can I use private GitHub repositories?](#can-i-use-private-github-repositories)
* [How do I import a subpath of a source?](#how-do-i-import-a-subpath-of-a-source)
* [How do I import NixOS modules](#how-do-i-import-nixos-modules)
* [Can I use local packages?](#can-i-use-local-packages)

### Can I use private GitHub repositories?

Yes. There are two ways:

#### 1. Use the git protocol

When using the git protocol, your public SSH keypair is used to authenticate
you:

``` shell
$ niv add git git@github.com:my_user/my_private_repo
```

##### 2. Use the netrc file

in order to `niv add` a private github repo you'll need to:

1. create a .netrc file with the following content
```
machine github.com
  login YOUR_GITHUB_USER_NAME
  password YOUR_GITHUB_TOKEN
```

2. add the path to the above file to `/etc/nix/nix.conf`:
```
netrc-file = /PATH/TO/.netrc
```

3. set `GITHUB_TOKEN` env var when calling `niv add`
```
GITHUB_TOKEN=$YOUR_GITHUB_TOKEN niv add ...
```

### How do I import a subpath of a source?

In order to use the directory `dir` of a `my-package`, use the following
pattern:

``` nix
let
  sources = import ./nix/sources.nix;
in sources.my-package + "/dir"
```

in this example, `sources.my-package` becomes `my-package`'s root directory, and `+ "/dir"` appends the
subdirectory.

### How do I import NixOS modules?

After the package containing the modules has been `niv add`ed, importing the
modules is straightforward:

``` nix
let
  sources = import ./nix/sources.nix;
in {
  imports = [ (sources.package + "/path/to/module") ];
}
```

### Can I use local packages?

If you need to use a local path as a source -- especially convenient when
modifying dependencies -- `niv` allows you to override the `sources.json` via
environment variables. To override a source `foo` with a local path
`./bar/baz`, set the environment variable `NIV_OVERRIDE_foo` to `./bar/baz`.

Generally, if the environment variable `NIV_OVERRIDE_<name>` is set _and_ you
have a source named `<name>` then `niv` will use the value of
`NIV_OVERRIDE_<name>` as the `outPath` of that source. All non-alphanumeric
characters in the source name are escaped to the character `_`; i.e. to
override the package `my package-foo` you need to set the environment variable
`NIV_OVERRIDE_my_package_foo`.
