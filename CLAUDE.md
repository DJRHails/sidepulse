## Patch-Stack Fork Workflow

This checkout is a patch-stack fork of
[inteliwear/sidepulse](https://github.com/inteliwear/sidepulse), managed by
[DJRHails/patch-stack-action](https://github.com/DJRHails/patch-stack-action).

| Branch | What it is |
| --- | --- |
| `base` | Read-only mirror of upstream `main`. Never commit to it. |
| `patch/*` | One logical change each, rebased onto `base` nightly. All real work happens here. |
| `main` | Rebuilt from scratch: `base` + `fork:` commits + each patch squash-merged as `patch-stack: …`. |

Nightly sync (04:00 UTC, `.github/workflows/patch-stack-sync.yml`): mirror `base`
from upstream, rebase every `patch/*` onto it (conflicts resolved by Claude Code),
rebuild `main`, and archive patches whose upstream PR was merged or closed.

Creating a patch:

```bash
git checkout -b patch/my-feature fork/base
# commit with normal conventional commits
git push fork patch/my-feature
gh pr create -R DJRHails/sidepulse --head patch/my-feature --base base
```

Dependent patches use `--` in the branch name: `patch/a--b` rebases onto `patch/a`.

Rules:

- Every `patch/*` branch needs a PR targeting `base`, never `main`.
- Never merge a patch into `main`, rebase `main`, or commit to `base` by hand.
- Fork-only infra on `main` must be prefixed `fork: ` or the next rebuild drops it.
- An open patch PR does not mean unintegrated; check `git log fork/main`.

Required repo secrets: `PATCH_STACK_APP_ID`, `PATCH_STACK_APP_PRIVATE_KEY`
(the patch-stack GitHub App), `CLAUDE_CODE_OAUTH_TOKEN` (`claude setup-token`).

Local install: the `sidepulse` CLI on trifle runs from
`~/.local/share/sidepulse/venv`. After `main` changes, reinstall from this fork's
`main` into that venv and restart the SidePulse service.
