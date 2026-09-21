# archdochq/lint

[![Use this action](https://img.shields.io/badge/GitHub_Marketplace-Use_this_action-2088FF?logo=github&logoColor=white)](https://github.com/marketplace/actions/archdoc-lint)

Runs [ArchDoc](https://github.com/archdochq/archdoc)'s lint over a
specification repository and puts every finding on the diff as an annotation.

```yaml
- uses: actions/checkout@v7
  with:
    fetch-depth: 0
- uses: archdochq/lint@v1
```

## Why not just run it

Because `archdoc lint` prints to a log, and a log is not where anyone reads a
finding. This runs `--json` and turns each one into a workflow annotation, so
`L08: missing required section "Motivation"` appears against the line of the
document that caused it, in the pull request, next to the change that caused
it.

It also does one thing a `run:` step cannot. A finding's path is relative to
`root`, and an annotation's must be relative to the workspace. With a `root`
of `docs`, or a specification in a subdirectory, those differ, and every
annotation would land on a file that does not exist. The action reads
`archdoc.json` and joins the two.

## Inputs

| Input | Default | |
| --- | --- | --- |
| `version` | *(empty)* | The archdoc release to use |
| `path` | `.` | The directory to run in, as `archdoc -C` takes it |
| `strict-warnings` | `false` | Treat warnings as failures |

Left empty, `version` uses whatever `archdoc` is already on `PATH` and
installs the latest release only if there is none. Given a value, that release
is installed whatever is already there, so an explicit request is never
silently ignored.

That default is what lets a workflow build the binary itself and keep it.
ArchDoc's own repository lints its specification with a binary built from the
commit under test rather than a downloaded release, because a spec change and
the code change it describes land together:

```yaml
- uses: actions/setup-go@v5
  with:
    go-version-file: go.mod
- run: go build -o /usr/local/bin/archdoc ./cmd/archdoc
- uses: archdochq/lint@v1      # finds it, installs nothing
  with:
    path: spec
```

## Outputs

| Output | |
| --- | --- |
| `findings` | How many findings were reported |

## Exit status

The action exits with `archdoc lint`'s own status: 0 when clean, 2 when there
are errors, or when there are warnings and `strict-warnings` is set. A usage
or configuration failure exits 1 and annotates nothing, because there is no
findings array to read.

`fetch-depth: 0` matters. The frozen-document rule compares every terminal
document against the branch, and a shallow checkout has no commits to compare
against.

## Licence

MIT. ArchDoc itself is AGPL-3.0-or-later; this action only runs a published
release and imposes nothing on what you run it against.
