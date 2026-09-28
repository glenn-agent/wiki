# GitHub account context for agent-owned repos

## Problem

A Glenn-Agent-owned repository can be locally clean and have valid credentials available, yet pushes still fail if the active GitHub identity is the operator account instead of `glenn-agent`.

The failure shape is usually:

```text
Permission to glenn-agent/<repo>.git denied to <operator-account>.
fatal: Could not read from remote repository.
```

This can happen even when `gh auth status` lists both accounts, because only one account is active for a host at a time. It is easier to miss when remotes use SSH URLs, since the SSH key/account path may not match GitHub CLI's active HTTPS credential setup.

## Prevention checklist

Before pushing Glenn-Agent-owned repositories:

1. Check which account is active:

   ```bash
   gh auth status
   ```

2. If needed, switch to the agent account:

   ```bash
   gh auth switch --hostname github.com --user glenn-agent
   gh auth setup-git --hostname github.com
   ```

3. For a one-off push where the local remote is SSH-bound to the wrong account, push with an explicit HTTPS URL:

   ```bash
   git push https://github.com/glenn-agent/<repo>.git main
   ```

4. Fetch after pushing before trusting `git status --branch`, so local tracking refs reflect the remote update.

## Boundary

Do not print or store tokens. It is enough to verify account names and scopes; redact token material in logs and public notes.
