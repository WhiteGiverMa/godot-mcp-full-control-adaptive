# Upstream Sync Workflow

This fork is maintained against:

- **origin**: `https://github.com/WhiteGiverMa/godot-mcp-full-control-adaptive`
- **upstream**: `https://github.com/tugcantopaloglu/godot-mcp`

## Why this fork exists

This fork carries local adaptations that are useful for automation-heavy Godot projects, especially the GUIDE-aware runtime input injection added in `mcp_interaction_server.gd`.

The goal is to keep these changes small, explicit, and easy to replay when upstream evolves.

## Recommended repository layout

Keep the fork as the **single source of truth**.

- Edit source in `src/scripts/`
- Run `npm run build`
- Consume generated runtime scripts from `build/scripts/`

For downstream Godot projects, prefer **vendoring only these built scripts**:

- `build/scripts/godot_operations.gd`
- `build/scripts/mcp_interaction_server.gd`

Do **not** blindly replace a whole `addons/godot_mcp/` directory if the project keeps local files such as:

- `plugin.cfg`
- `mcp_editor_plugin.gd`
- project-local README / AGENTS files

## Normal update flow

### 1. Fetch upstream

```powershell
git fetch upstream
git fetch origin
```

### 2. Rebase or merge upstream/main

If local fork changes are still linear and small, prefer rebase:

```powershell
git checkout main
git rebase upstream/main
```

If rebase becomes noisy, merge is acceptable:

```powershell
git checkout main
git merge upstream/main
```

## Conflict resolution policy

When conflicts happen, treat these files as high-value local adaptation points:

- `src/scripts/mcp_interaction_server.gd`
- `README.md`
- any future tests/docs that mention GUIDE-aware injection

Re-verify the following behavior after conflict resolution:

1. `action="jump"` style runtime input resolves GUIDE bindings when GUIDE is active
2. raw key injection still works
3. fallback to `Input.action_press()` still works when no GUIDE binding is found

## Rebuild after sync

Always rebuild after changing source files:

```powershell
npm install
npm run build
```

## Test after sync

Run the full test suite before pushing:

```powershell
npm test
```

If the fork changes runtime input behavior, also do a live project verification against a Godot project that uses GUIDE.

## Downstream project sync

For downstream Godot projects, update by copying the built scripts:

```powershell
Copy-Item "G:\dev\godot-mcp-fc-a\build\scripts\godot_operations.gd" "<project>\addons\godot_mcp\godot_operations.gd" -Force
Copy-Item "G:\dev\godot-mcp-fc-a\build\scripts\mcp_interaction_server.gd" "<project>\addons\godot_mcp\mcp_interaction_server.gd" -Force
```

Then verify hashes or diffs, and run the target project once.

## Push flow

After sync and verification:

```powershell
git status
git add .
git commit -m "<message>"
git push origin main
```

## Design rule for future local changes

Keep fork-specific behavior:

- narrowly scoped
- documented in README or docs
- validated in a real Godot project when runtime behavior changes

If a change is generic enough for everyone, consider upstreaming it later.
