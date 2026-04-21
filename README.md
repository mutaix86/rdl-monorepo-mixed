# rdl-monorepo-mixed

Test fixture for the `remove_duplicate_lockfiles` maintenance task.

## Scenario

A monorepo with **three** independent lockfile conflicts spanning two ecosystems, exercising both auto-detect branches and the ambiguous branch in a single run.

```
/package.json                   (workspace root)
/pnpm-workspace.yaml            ← pnpm signal
/package-lock.json              ← npm lockfile
/pnpm-lock.yaml                 ← pnpm lockfile
/packages/api/pyproject.toml    ([tool.uv])
/packages/api/poetry.lock
/packages/api/uv.lock
/packages/web/package.json      (no packageManager, no config)
/packages/web/yarn.lock
/packages/web/bun.lock
```

## Expected MT behaviour

| Directory       | Ecosystem | Signal                    | Resolution           |
|-----------------|-----------|---------------------------|----------------------|
| `.` (root)      | node      | `pnpm-workspace.yaml`     | **Auto** → keep pnpm, delete `package-lock.json` |
| `packages/api`  | python    | `[tool.uv]` in pyproject  | **Auto** → keep uv, delete `poetry.lock`          |
| `packages/web`  | node      | (none)                    | **Asks user**: options `["bun", "yarn"]`          |

- One select question is raised (for `packages/web` node), plus the `update_gitignore` boolean.
- `.gitignore` at repo root records all three deleted files (paths relative to repo root).
