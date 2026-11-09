# renovate-devengines-repro

Minimal reproduction for
[renovatebot/renovate#46989](https://github.com/renovatebot/renovate/discussions/46989).

## Setup

`renovate.json` only extends `config:js-app`, which includes
`:pinAllExceptPeerDependencies`.

`package.json` declares a range in each of these sections:

| Section                      | Value                   | Expected   | Actual     |
| ---------------------------- | ----------------------- | ---------- | ---------- |
| `dependencies`               | `react: ^19.3.1`        | pinned     | pinned     |
| `devDependencies`            | `typescript: ^7.0.2`    | pinned     | pinned     |
| `peerDependencies`           | `react-dom: ^19.3.1`    | range kept | range kept |
| `engines`                    | `node: >=18.0.0`        | range kept | range kept |
| `devEngines.runtime`         | `node: ^22.18.0 \|\| >=24.0.0` | range kept | **pinned** |
| `devEngines.packageManager`  | `npm: ^10.0.0 \|\| ^11.0.0`   | range kept | **pinned** |

## Current behavior

Renovate's "Pin dependencies" PR pins `dependencies` and `devDependencies`
and leaves `peerDependencies` and `engines` alone, but it also pins
`devEngines.runtime` and `devEngines.packageManager` to a single version.

See [#1](https://github.com/JstnMcBrd/renovate-devengines-repro/pull/1)
for the "Pin dependencies" PR that Renovate opened in this repository, which
pins both `devEngines` ranges.

## Expected behavior

`devEngines` declares the supported runtime and package manager versions, just
like `engines`, so its ranges should be kept.

## Cause

`:pinAllExceptPeerDependencies` only exempts the `engines` and
`peerDependencies` dep types. The `devEngines.runtime` and
`devEngines.packageManager` dep types are not exempted, so the
`matchPackageNames: ["*"]` pin rule applies to them.
