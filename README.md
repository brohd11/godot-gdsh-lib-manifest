# godot-gdsh-lib-manifest

The reviewed index of gdsh command packages that installers serve:
`mcp_sh_godot gdsh install NAME` today, and `gdsh_cli package install` later. Packages land in a
command root (`~/.gdsh/packages/<name>`, `~/.mcp-sh/godot/packages/<name>` or a project's),
with their dependencies installed as siblings.

## Trust model

Each entry pins a `tag` and the `commit` it was reviewed at. Installs download that commit, so
nothing reaches users until a person moves the pin, and a moved tag can't change what is
served. Only pinned packages can be installed by name.

## Entries

```json
{
  "name": "terrain3d",                       // install folder; must match package.cfg path's basename
  "repo": "brohd11/godot-gdsh-lib-terrain3d",
  "summary": "...",                          // shown by `gdsh search`
  "tags": ["3d", "terrain"],
  "tag": "v0.1.1",                           // the reviewed pin...
  "commit": "<sha>",                         // ...and the commit it pointed at
  "gdsh": "v0.4.5",                          // derived: minimum gdsh
  "requires": ["brohd11/godot-gdsh-lib-level"], // derived: indexed gdsh packages
  "project_only": []                         // derived: non-gdsh requires (refused in command roots)
}
```

You write `name`, `repo`, `summary`, `tags` and the pin. The derived fields come from
`package.cfg` at the pinned commit. A package qualifies when it has `cmd/manifest.gd` and makes
no absolute `res://addons/_lib/gdsh...` references outside it, because those folders don't
exist in a command root. Find dependencies as siblings instead.

## Maintaining

```sh
go install github.com/brohd11/gdsh-pkg/cmd/gdsh-manifest@latest
gdsh-manifest check            # what CI runs; -w rewrites derived fields
gdsh-manifest pin level v0.2.0 # after reviewing the package at that tag
gdsh-manifest propose          # newer tags than the pins
```

- To add a package, write its entry with `tag`/`commit` empty, run `gdsh-manifest pin NAME TAG`,
  and open a PR.
- `check.yml` runs the check on every push and PR.
- `propose.yml` runs weekly. It opens one PR per newer tag, with a compare link, and never
  merges.
