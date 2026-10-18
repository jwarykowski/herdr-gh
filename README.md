<p align="center"><img src="assets/herdr-gh.svg" alt="herdr-gh" width="120"></p>

# herdr-gh

A [herdr](https://herdr.dev) plugin that puts each workspace's pull request in
the sidebar — number, CI and review in a few characters, next to the branch:

```
◑ api-auth-fix
  fix/auth-expiry #412│×│!
✓ docs-update
  docs/readme #415│√│+
● search
  feat/search d418│•
  old/cleanup #401│↗
```

| part | meaning |
|---|---|
| `#412` / `d412` | PR number (`d` = draft) |
| `√` `×` `•` | CI passed, failed, running |
| `+` `!` | review approved, changes requested (blank while waiting) |
| `↗` | merged — time to remove the worktree |
| `†` | closed without merging |

Workspaces whose branch has no PR show nothing. Symbols are deliberately
plain so most monospace fonts have them; all of them are configurable.

## Requirements

- herdr 0.9.3+
- [`gh`](https://cli.github.com), logged in (`gh auth login`)
- `jq`, `git`, `bash`

## Install

```sh
herdr plugin install jwarykowski/herdr-gh
```

Then add the `$gh` token to the sidebar in herdr's `config.toml` (this is the
default layout plus `$gh`):

```toml
[ui.sidebar.spaces]
rows = [["state_icon", "workspace"], ["branch", "git_status", "$gh"]]
```

The reporter starts with each herdr session, polls every minute, and exits
when herdr stops. Values expire after three missed polls, so nothing stale is
left behind.

## Actions

| action | does |
|---|---|
| `jwarykowski.herdr-gh.refresh` | report now instead of waiting for the next poll |
| `jwarykowski.herdr-gh.restart` | restart the reporter, e.g. after a config change |

Bind one if you like:

```toml
[[keys.command]]
key = "prefix+alt+r"
type = "plugin_action"
command = "jwarykowski.herdr-gh.refresh"
description = "refresh GitHub status"
```

## Hooks

Run your own commands when a PR changes: an executable in
`$(herdr plugin config-dir jwarykowski.herdr-gh)/hooks/` named after the event.
The names are [keybars Pro](https://keybars.app/pro/cli/#event-names)'s GitHub
events, so a hook can pass them straight on.

| event | when |
|---|---|
| `pull_request.merged` / `pull_request.closed` | the PR was merged / closed without merging |
| `ci.running` | checks started (again) |
| `ci.passed` / `ci.failed` | checks finished |
| `review.approved` / `review.changes_requested` | the review decision changed |

Changes are spotted by the poll, so a hook runs up to a minute after the
event. A PR the reporter hasn't seen before is only recorded, so restarts and
branches merged while herdr was closed fire nothing.

Hooks get `HERDR_GH_EVENT`, `_PR` (number), `_URL`, `_REPO` (`owner/name`),
`_TITLE`, `_BRANCH` and `_WORKSPACE_ID`, and log to `hooks.log` in the
plugin's state directory. For example, `hooks/pull_request.merged` sends the
event to keybars Pro, where any GitHub rule, such as the marketplace's
merge-confetti, plays it:

```sh
#!/bin/sh
keybars-pro notify -source github -repo "$HERDR_GH_REPO" -branch "$HERDR_GH_BRANCH" "$HERDR_GH_EVENT"
```

[herdr-keybars](https://code.warykowski.io/jwarykowski/herdr-keybars)'s setup
links all of them for you.

## Configuration

Optional. Put overrides in `config.env` inside the plugin config directory
(`herdr plugin config-dir jwarykowski.herdr-gh`), then run the restart action.
See [`config.env.example`](config.env.example) for every setting.

## How it works

For every workspace the reporter takes the first pane's directory, asks
`gh pr view` about the checked-out branch, and reports the result with
`herdr workspace report-metadata` under the source `herdr-gh`. That is about
one GitHub API call per workspace per poll.

## Support

herdr-gh is free and MIT licensed. If it saves you a trip to the browser, you
can [sponsor its development](https://github.com/sponsors/jwarykowski).

## License

MIT
