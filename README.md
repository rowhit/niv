# niv

A tool for dealing with third-party packages in [Nix].

## Building

Inside the provided nix shell:

``` bash
$ # GHCi:
$ snack ghci
$ # actual build:
$ snack build
```

## Usage

**NOTES**

* no support for non-json, to enforce convention
* fixed path to nix/versions.json, to enforce convention

### Commands

Abbreviations:

### Attributes

* `-b` -> `--branch`
* `-n` -> `--name`
* `-o` -> `--owner`
* `-r` -> `--repo`
* `-t` -> `--template`
* `-a` -> `--attribute`

### VCS

* `-h` -> `--github`
* `-l` -> `--gitlab`

#### init

* `[<p1> --branch foo <p2> ...]`

Creates (if the file doesn't exist)

* `nix/versions.json`:
``` json
{"nixpkgs": { ... }}
```

*`nix/fetch.nix`:
``` nix
...
```

*`default.nix`:
``` nix
with { fetch = import <fetch>; };
let pkgs = import fetch.nixpkgs;
in pkgs.hello
```

#### add

* `<package>`: adds the following to the versions file where `let <username/repo> = <package>`
``` json
{ "<repo>":
  { "owner":  "<username>",
    "repo":   "<repo>",
    "rev":    "<latest commit on <branch>>",
    "sha256": "<sha256>",
    "branch": "<branch>"
  }
}
```

* `--branch`: specifies `<branch>` (default: master)
* `--username <username>`: then `let <repo> = <package>`
* `--gitlab`: use gitlab instead of GitHub
* `--attribute <attribute> <value>`: sets `<attribute>` to `<value>`

If the package already exists, merges with the package (prior to heuristics)

#### update

* `[p [--commit] [--branch]]`
 - `[]`: all packages are updated
 - `[p1 p2 ...]`: the specified packages are updated

* `--commit <rev>`: `rev` is set to `<rev>` and the package is prefetched
* `--branch <branch>`: `branch` is set to `<branch>`, `rev` is set to the
  latest revision on that branch and the package is prefetched

#### show

* Shows all packages

#### drop

`<p1> <p2>`

* Drops the specified packages

**NOTE**: should the URLs be used instead? or more simply, how do we differentiate between Gitlab/GitHub?

[Nix]: https://nixos.org/nix/
