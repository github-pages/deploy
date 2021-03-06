# primer/deploy

This [GitHub Action][github actions] deploys to [Now] and aliases the successful deployment to a predictable URL according to the following conditions:

1. We run `now` without any arguments to get the "root" deployment URL, which is generated by Now.
1. If the branch is `master`, we treat the `alias` field in `now.json` as the production URL and:
    1. If there is a `rules.json`:
        * `now alias <deployment> <name>.now.sh` to create a fallback URL for path aliases
        * `now alias <deployment> <production> -r rules.json` to set up path aliases
    1. `now alias <deployment> <production>` to alias the production URL.
1. `now alias <deployment> <name>-<branch>.now.sh` to alias the root deployment to a branch-specific URL.

The app name (`<name>`) and branch (`<branch>`) are both "slugified" to strip invalid characters so that they'll work as URLs. Leading non-word characters are removed, and any sequence of characters that isn't alphanumeric or `-` is replaced with a single `-`. In other words, `@primer/css` becomes `primer-css`, `shawnbot/some_branch` becomes `shawnbot-some-branch`, and so on.

## Status checks
Two [status checks] will be listed for this action in your checks: **deploy** is the action's check, and **deploy/alias** is a [commit status] created by the action that reports the URL and links to it via "Details":

![image](https://user-images.githubusercontent.com/113896/52000881-f8c45980-2472-11e9-8d04-00264094437b.png)

**Note:** Checks listed in the PR merge box (above) always point to the most recent commit, but you can access the list of [checks][status checks] for the last commit of every push by clicking on the status icon (usually a ![green check](https://user-images.githubusercontent.com/113896/52001573-99674900-2474-11e9-82ab-6414e3f004cf.png) or ![red x](https://user-images.githubusercontent.com/113896/52001543-88b6d300-2474-11e9-84ca-82ff51828ea9.png)) in your repo's "Commits" and "Branches" pages, or commit history on a PR page:

![image](https://user-images.githubusercontent.com/113896/52001489-64f38d00-2474-11e9-92ea-827e466eb948.png)

## Usage
To use this action in your own workflow, add the following snippet to your `.github/main.workflow` file:

```hcl
action "deploy" {
  uses = "primer/deploy@master"
  secrets = [
    "GITHUB_TOKEN",
    "NOW_TOKEN",
  ]
}
```

**You will need to provide a [Zeit token](https://zeit.co/account/tokens) value for the `NOW_TOKEN` secret in the Actions visual editor** if you haven't already.

To avoid racking up failed deployments, we suggest that you place this action after any linting and test actions.

## Now CLI arguments
It's possible to pass additional arguments through to the `now` CLI via the `args` field in your workflow action. Because the `primer-deploy` CLI accepts options of its own (such as `--dry-run`), you need to prefix any `now` arguments with `--`:

```diff
action "deploy" {
  uses = "primer/deploy@master"
+  args = "-- --meta autoDeployed=true"
```

You can also use `args` to deploy a subdirectory, e.g. `docs`:

```diff
action "deploy" {
  uses = "primer/deploy@master"
+  args = "-- docs"
```


[now]: https://zeit.co/now
[github actions]: https://github.com/features/actions
[commit status]: https://developer.github.com/v3/repos/statuses/
[status checks]: https://help.github.com/articles/about-status-checks/

