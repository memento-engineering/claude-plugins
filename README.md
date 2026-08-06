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
| `leonard` | [`lenny`](https://github.com/memento-engineering/lenny) | The `drive-with-leonard` skill plus the `leonard-driver` and `leonard-pilot` agents — drive a running Flutter app (or any Dart-VM target with a Leonard extension) with an LLM over the VM service. |

## How this catalog is built

This repository holds **only** `.claude-plugin/marketplace.json`. It ships no
skills, no agents, and no copies of anything.

Each entry uses `strict: false` and points its `skills` / `agents` fields at
paths inside the repository that already owns those assets. For `leonard` that
is `packages/leonard_cli/lib/assets/`, the same tree that
`dart run leonard_cli:install` vends into `.agents/`. So a skill has exactly one
source of truth, and a plugin user and an `install` user read the same file.

Three consequences worth knowing:

- **New skills join automatically.** The `skills` field names a *directory*, and
  Claude Code scans it for `<name>/SKILL.md`. Adding a skill to the source repo
  adds it to the plugin with no change here.
- **New agents do NOT.** `agents` accepts individual **files** only — a
  directory path fails validation with `plugins.N.agents: Invalid input`. Every
  agent must be listed explicitly, so adding one to a source repo also means
  adding a line here. (The agent's name comes from its `name:` frontmatter, not
  its filename.)
- **Keeping the catalog small is the point.** Adding a marketplace clones the
  repository that hosts it. Hosting this JSON inside `lenny` would pull ~62 MB
  (the tracked DevTools web bundles) just to read the catalog. Here, that clone
  only happens if someone installs the plugin.

## Versioning

Plugin entries have no `version`, so each resolves to the source repo's current
commit SHA on `main` — users track HEAD. To pin a plugin to a release instead,
add `"version": "0.3.0"` to its entry and bump it deliberately; users then only
see an update when that string changes.

## Adding a plugin

Append an entry to `plugins` in `.claude-plugin/marketplace.json`. `name` and
`source` are the only required fields. Then:

```sh
claude plugin validate .
```

The validator checks the manifest for schema errors, duplicate plugin names, and
source path traversal.

If you ever rename or remove an entry, add a top-level `renames` map from the
old name to the new one (or to `null` if it's gone) so existing installs migrate
instead of erroring with `plugin-not-found`.

## License

MIT — see [LICENSE](LICENSE). The plugins themselves are licensed by their own
repositories.
