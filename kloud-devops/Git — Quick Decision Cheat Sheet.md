#git #linux 


> **Mental model:**
> 
> `pull` = get → `push` = send → `rebase` = replay → `stash` = park → `reset` = discard → `revert` = undo safely → `squash` = combine → `conflict` = resolve → `hook` = automate

## Mindmap

```mermaid
mindmap
  root((Git))
    Remote
      pull
        "Remote → Local"
      push
        "Local → Remote"
      force-with-lease
        "After rebase"
        "Rewritten history"

    History
      rebase
        "Feature + latest master"
        "Linear history"
        "Replay commits"
      squash
        "Combine commits"
        "Clean history"
      revert
        "Undo shared commit"
        "Creates new commit"
      reset --hard
        "Discard uncommitted changes"

    Temporary
      stash
        "Save unfinished work"
        "Switch branch"
        "stash pop"

    Conflicts
      resolve
        "Fix conflict manually"
        "git add"
        "continue / commit"

    Hooks
      "Automate Git actions"
      "pre-commit"
      "post-update"
      "Run scripts automatically"
```

## Scenario → Command

|Scenario|Command / Action|Remember|
|---|---|---|
|Need latest remote changes|`git pull`|**Get**|
|Send committed work|`git push`|**Send**|
|Feature needs latest `master`|`git rebase master`|**Replay**|
|Rebased branch → remote|`git push --force-with-lease`|**Rewrite safely**|
|Need to switch with unfinished work|`git stash`|**Park**|
|Delete uncommitted tracked changes|`git reset --hard HEAD`|**Destroy**|
|Undo a shared/bad commit|`git revert <commit>`|**New undo commit**|
|Combine multiple commits|`git rebase -i HEAD~N` → `squash`|**Combine**|
|Merge/rebase conflict|Fix → `git add` → continue/commit|**Resolve**|
|Automate action after Git event|Git hook|**Automate**|

## 7-Second Decision Tree

```mermaid
flowchart TD
    A{"What's the problem?"}
    A -->|Need remote changes| B["git pull"]
    A -->|Send commits| C["git push"]
    A -->|Feature needs latest master| D["git rebase master"]
    A -->|Rebased branch needs pushing| E["git push --force-with-lease"]
    A -->|Save unfinished work| F["git stash"]
    A -->|Discard local changes| G["git reset --hard HEAD"]
    A -->|Undo shared commit| H["git revert <commit>"]
    A -->|Combine commits| I["git rebase -i HEAD~N → squash"]
    A -->|Conflict| J["Fix → git add → continue/commit"]
    A -->|Automate Git action| K["Git hook"]
```

## Memory Hook

```text
PULL              → Get
PUSH              → Send
REBASE            → Replay
SQUASH            → Combine
STASH             → Park
RESET             → Destroy
REVERT            → Undo
FORCE-WITH-LEASE  → Safely rewrite remote
CONFLICT          → Fix → Add → Continue
HOOK              → Automate
```

### ⭐ Critical Distinctions

```text
Uncommitted + KEEP       → git stash
Uncommitted + DELETE     → git reset --hard

Committed + SHARED + UNDO → git revert
Feature + latest master   → git rebase
Rebased + PUSH            → git push --force-with-lease

Many commits + CLEAN      → squash
Git can't merge changes   → resolve conflict manually
Git event + automation    → hook
```