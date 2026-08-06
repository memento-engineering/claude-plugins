# memento-plugins

A Claude Code plugin marketplace for the [memento.engineering](https://github.com/memento-engineering)
toolchain.

```
/plugin marketplace add memento-engineering/claude-plugins
/plugin install leonard@memento-plugins
```

## What's in it

| Plugin | Source repo | What you get |
| --- | --- | --- |
| `leonard` | [`lenny`](https://github.com/memento-engineering/lenny) | Drive and test a running Flutter app — or any Dart-VM target with a Leonard extension — with an LLM over the VM service. |

Deliberately not enumerated: the exact skills and agents each plugin carries.
That list lives in the source repo and changes there; repeating it here creates a
second copy to keep in sync, which is the failure this catalog is built to avoid.
Run `/plugin` after installing to see what a plugin actually exposes.

## How this catalog is built

This repository holds **only** `.claude-plugin/marketplace.json`. It ships no
skills, no agents, and no copies of anything.

Each entry roots its plugin at the **vendored asset directory** of the repo that
owns it, using a `git-subdir` source. For `leonard` that is
`packages/leonard_cli/lib/assets/` in `lenny` — the same tree that
`dart run leonard_cli:install` vends into `.agents/`. So a skill has exactly one
source of truth, and a plugin user and an `install` user read the same file.

### Root the plugin at the vendored surface, never at the repo root

**This is the rule that matters.** A plugin's skills load from `skills/` under
its source root, and a marketplace entry's `skills` field *adds to* that scan
rather than replacing it. (The "listed paths are the complete set" behaviour
applies only to a marketplace-root source, `"source": "./"`.)

`lenny` has `skills` and `agents` symlinks at its repo root, tracked in git,
pointing into `.agents/` — ten repo-internal development skills (`beads`,
`critique`, `debug-inference`, `deliberate`, `discover`, `factory`,
`factoryskills`, `forge`, `predictable-flutter`, `specify`) and two internal
agents. A `github` source rooted at the repo root therefore ships **all of them**
to every consumer, no matter what the entry's `skills` field says.

`git-subdir` fixes it at the root rather than by filtering: the plugin root
becomes `packages/leonard_cli/lib/assets/`, which contains nothing but the
consumer surface. Anything not deliberately vendored there is unreachable, so
this can't regress by accident. It also clones sparsely, so installing pulls the
asset directory instead of `lenny`'s ~62 MB of tracked DevTools web bundles.

Two consequences worth knowing:

- **New skills join automatically.** `./skills/` names a *directory*, scanned for
  `<name>/SKILL.md`. Vendor a skill into the source repo's asset tree and it
  appears in the plugin with no change here.
- **New agents do NOT.** `agents` accepts individual **files** only — a directory
  path fails validation with `plugins.N.agents: Invalid input`. Every agent must
  be listed explicitly, so vendoring one also means adding a line here. (The
  agent's name comes from its `name:` frontmatter, not its filename.)

## Versioning

Plugin entries have no `version`, so each resolves to the source repo's current
commit SHA on `main` — users track HEAD. To pin a plugin to a release instead,
add `"version": "0.3.0"` to its entry and bump it deliberately; users then only
see an update when that string changes.

## Adding a plugin

Append an entry to `plugins` in `.claude-plugin/marketplace.json`. `name` and
`source` are the only required fields. Point the source at a directory that holds
**only** what you intend to ship — see the rule above. Then:

```sh
claude plugin validate .
```

The validator checks the manifest for schema errors, duplicate plugin names, and
source path traversal. It does **not** fetch the source, so it cannot tell you
what a plugin actually exposes. Before publishing an entry, list the files under
its source path and confirm every one of them belongs in a consumer's context.

If you ever rename or remove an entry, add a top-level `renames` map from the
old name to the new one (or to `null` if it's gone) so existing installs migrate
instead of erroring with `plugin-not-found`.

## License

MIT — see [LICENSE](LICENSE). The plugins themselves are licensed by their own
repositories.
