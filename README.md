# Rebase Bot

Rebase Bot is a tool that allows you to synchronize code between repositories using `git rebase` command and then create a PR in GitHub. The work is based on ShiftStack's [merge bot](https://github.com/shiftstack/merge-bot/tree/main/src/merge_bot).

# Usage

## Dest and Source parameters
The bot takes a desired branch in `dest` repository and rebases it onto a branch in the `source` repository.
The `source` can be any git repository, but `dest` must belong to GitHub. Therefore the format for `--source` and `--dest` is slightly different.

```txt
...
--source <full_repository_address>:<branch_name> \
--dest <github_org>/<repository_name>:<branch_name> \
...
```

## Rebase parameter

To successfully create a PR we need an intermediate repository where we push our changes first. That's why `--rebase` option is required. The format is the same as for `--dest` option. It could be any repository, from your private one to a repository in another GitHub organization.

```txt
...
--rebase <github_org>/<repository_name>:<branch_name> \
...
```

## Auth parameters

There are two auth modes that the bot supports: `user` and `application`. Therefore different parameters can be provided for the bot.

### User credentials mode

In this mode you will provide your own GitHub token and the bot will do the work on your behalf.

In this case you need to provide just one parameter `--github-user-token` where the value is a path to the file with your GitHub token:

```txt
...
--github-user-token /path/to/github_access_token.txt \
...
```

### Application credentials mode

Before using the application mode you need to create 2 GitHub applications: `app` to create resulting PRs, and `cloner` which will push changes in the intermediate repository, specified by `--rebase`.

`app` should be installed in the `dest` GitHub organization with the following permissions:

    - Contents: Read
    - Metadata: Read-only
    - Pull requests: Read & Write

`cloner` application is to be installed in the `rebase` GitHub organization with the permissions as follows:

    - Contents: Read & Write
    - Metadata: Read-only
    - Workflows: Read & Write

Here are instructions on how to [create](https://docs.github.com/en/developers/apps/building-github-apps/creating-a-github-app) and [install](https://docs.github.com/en/developers/apps/managing-github-apps/installing-github-apps) a GitHub application.


When both applications are successfully installed, you need to download their private keys and store in a file on a local disk.

To perform this work on behalf of the applications, the bot needs their private keys specified be `--github-app-key` and `--github-cloner-key` parameters. Both should contain paths to to the corresponding private keys.

```txt
...
--github-app-key /path/to/app-private-key.pem \
--github-cloner-key /path/to/cloner-private-key.pem \
...
```

Then the bot needs application IDs, which are presented as 6-digit numbers:

```txt
...
--github-app-id <6-digit number> \
--github-cloner-id <6-digit number> \
...
```

## Optional parameters

### Dry run

If you don't want to create a PR, but just to perform a rebase locally, you can set `--dry-run` flag. In this case the bot will stop working right after the rebase.

### Custom rebase directory

By default the bot clones everything in `.rebase` folder. You can specify another working dir location with `--working-dir` option.

### Golang vendor update

It's useful only with Golang repositories, which require a `vendor` folder with all dependencies. If `--update-go-modules` flag is set, then the bot will create another commit on top of the rebase, which contains changes in the `vendor` folder.

### Slack Webhook

If you want to be notified in Slack about the status of recent rebases, you can set ``--slack-webhook` option. The value here is the path to a local file with the webhook url.

```txt
...
--slack-webhook /path/to/slack_webhook.txt \
...
```

### Custom git username and email

By default the bot takes global git username and email to perform the rebase. If you want to change it to something else you can use `--git-username` and `--git-email` options.

```txt
...
--git-username rebasebot \
--git-email rebasebot@example.com \
...
```

## Examples of usage

Example 1. Sync kubernetes/cloud-provider-aws with openshift/cloud-provider-aws using applications credentials. 

```sh
rebasebot --source https://github.com/kubernetes/cloud-provider-aws:master \
          --dest openshift/cloud-provider-aws:master \
          --rebase openshift-cloud-team/cloud-provider-aws:rebase-bot-master \
          --update-go-modules \
          --github-app-key ~/app.2021-09-10.private-key.pem \
          --github-app-id 137509 \
          --github-cloner-key ~/Dropbox/cloner.2021-09-10.private-key.pem \
          --github-cloner-id 137497 \
          --git-username cloud-team-rebase-bot --git-email cloud-team-rebase-bot@redhat.com 
```

Example 2. Sync kubernetes/cloud-provider-azure and openshift/cloud-provider-azure with user credentials.

```sh
rebasebot --source https://github.com/kubernetes/cloud-provider-azure:master \
          --dest openshift/cloud-provider-azure:master \
          --rebase openshift-cloud-team/cloud-provider-azure:rebase-bot-master \
          --update-go-modules \
          --github-user-token ~/my-github-token.txt
```
