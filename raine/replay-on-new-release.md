# Replay Raine Changes On A New Release

This repo should be maintained on top of upstream release tags, not `dev`.

Current maintained branch pattern:

- `raine/v<current-release>`

## Goal

When a new upstream release appears, create a new branch from that tag and replay Raine-specific commits onto it.

Example target:

- new tag: `v<next-release>`
- new branch: `raine/v<next-release>`

## Remotes

Assume:

- `upstream` = `https://github.com/anomalyco/opencode`
- `origin` = Raine's fork

Always fetch tags first:

```bash
git fetch upstream --tags
```

## Source Branch

The currently maintained branch should follow this naming pattern:

```bash
raine/v<current-release>
```

Example:

```bash
raine/v<current-release>
```

## Replay Workflow

1. Identify the current maintained branch and the base release tag it was created from.

Example:

```bash
current_branch="raine/v<current-release>"
current_tag="v<current-release>"
new_tag="v<next-release>"
new_branch="raine/v<next-release>"
```

2. Inspect the current maintained branch history relative to its base tag.

```bash
git log --oneline "$current_tag..$current_branch"
```

This range is the set of fork-only commits to replay.

The branch history is the source of truth. Do not maintain a separate canonical commit list in this file. Always derive the replay set from the current maintained branch and its base release tag.

3. Create the new branch from the new release tag.

```bash
git checkout -b "$new_branch" "$new_tag"
```

4. Cherry-pick the fork-only commits in order from oldest to newest.

Inspect them first:

```bash
git log --reverse --oneline "$current_tag..$current_branch"
```

Replay them one by one.

For normal commits:

```bash
git cherry-pick <commit>
```

For merge commits:

```bash
git cherry-pick -m 1 <merge-commit>
```

Notes:

- replay in the original order to minimize conflicts
- if the range contains a merge commit, use `-m 1` for that commit
- if a commit is fork maintenance only and no longer relevant on the new release, skip it intentionally and document why

## Known Compatibility Note

The original `add keybind for skills search` commit had to be fixed when it was first replayed onto a release branch.

The correct implementation is:

- `packages/opencode/src/config/keybinds.ts` contains:

```ts
prompt_skills: z.string().optional().default("<leader>k").describe("Open skills search")
```

- do not reintroduce broken edits into `packages/opencode/src/config/config.ts`

If the cherry-pick applies incorrectly on a future release, keep the prompt command wiring and docs change, then add only the minimal schema entry in the correct keybind config file for that release.

## Verification

After replaying, verify the branch builds and installs:

```bash
./raine/install-local.sh
```

Then verify the installed binary:

```bash
which opencode
opencode --version
```

## If Conflicts Happen

Use this rule:

- prefer the new release's file structure
- replay only the minimal Raine-specific behavior
- avoid copying large old config blocks into newer files

If a commit no longer makes sense on the new release, document that and skip or rewrite it as a small clean replacement commit.
