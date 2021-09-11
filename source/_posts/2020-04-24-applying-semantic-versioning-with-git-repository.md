---
title: Applying semantic versioning with git repository
date: 2020-04-24 15:35:00 Z
categories: [SemanticVersioning]
tags: [git, semantic-release]
cover: /images/applying-semantic-versioning-with-git-repository/featured.png
---

Semantic versioning is the practice of assigning version numbers based on the severity of the change.

<!-- more -->

For semantic versioning, the [Semver](https://semver.org/) used as a versioning standard, and 
it based on the [Conventional Commits](https://www.conventionalcommits.org/). 

By using the node library: [semantic-release](https://github.com/semantic-release/semantic-release) we can
automate the whole package release workflow including: determining the next version number, generating the release notes and publishing the package.

<div class="row">
<div class="col-6">
<img src="/images/applying-semantic-versioning-with-git-repository/semver.png" style="padding-top: 30px;">
</div>
<div class="col-6">
<img src="/images/applying-semantic-versioning-with-git-repository/SemanticVersioning.png" height="250">
</div>
</div>

In this post, I will explain how we can integrate semantic versioning in the git repository.


## Conventional Commits
To be used, semantic release needs commit message to follow a precise pattern. Here is the expected syntax for commit messages.

```
<type>(<scope>): <subject>

[optional body]

[optional footer(s)]
```

The commit contains the following structural elements, to communicate intent to the consumers of your library:

- `fix` ➜ patches a bug in your codebase `[PATCH]`
- `feat` ➜ introduces a new feature to the codebase `[MINOR]`
- `BREAKING CHANGE` ➜ (a commit that has a footer `BREAKING CHANGE:`, or appends a `!` after the type/scope) introduces a breaking API change `[MAJOR]`
- `build`, `chore`, `ci`, `docs`, `style`, `refactor`, `revert`, `perf`, `test` ➜ other types than `fix` and `feat`
- footers other than `BREAKING CHANGE: <description>` may be provided and follow a convention similar to [git trailer format](https://git-scm.com/docs/git-interpret-trailers)
- `<subject>` - succinct description of the change, use the imperative, present tense: `change` not `changed` nor `changes`, don’t capitalize first letter, no dot (.) at the end
- `<scope>` - We can insert as scope references to our JIRA/GITHUB/GITLAB issues id(XXX-1223)

## Enforce Conventional Commits
The automated versioning only work fully when all the team respect these rules. 
To ensure automated versioning, we need to add some custom hooks in your versioning system.

### Remote hooks setup on Github or Gitlab  

- `Github:` [Configure pre-receive hooks for an organization](https://help.github.com/en/enterprise/2.18/admin/developer-workflow/managing-pre-receive-hooks-on-the-github-enterprise-server-appliance#configure-pre-receive-hooks-for-an-organization)
- `Gitlab:` [Set a global server hook for all repositories](https://docs.gitlab.com/ce/administration/server_hooks.html#set-a-global-server-hook-for-all-repositories)

### Local hooks setup for the conventional commit:

- Run following commands in git repository directory
```    
echo "module.exports = {extends: ['@commitlint/config-conventional']};" > commitlint.config.js
echo '{"hooks":{"commit-msg":"commitlint -E HUSKY_GIT_PARAMS"}}' > .huskyrc
```
- Install `commitlint` and `husky` :
```    
npm init
npm install --save-dev husky @commitlint/{cli,config-conventional}
```
- Next `git commit` if you don't follow conventional commit patterns then it will reject 
the commit.
  
## Semantic Release  

- Install node library: [semantic-release](https://github.com/semantic-release/semantic-release)
```
npm i -g semantic-release @semantic-release/{git,exec,changelog}
```

- Create `.releaserc` in your repository
```
{
	"branches": ["master"],
	"plugins": [
        "@semantic-release/commit-analyzer",
        "@semantic-release/release-notes-generator",
        "@semantic-release/github",
        "@semantic-release/git",
        "@semantic-release/changelog",
        ["@semantic-release/exec", {
              "prepareCmd" : "echo '${nextRelease.version} run your custom shell script'",
              "publishCmd" : "echo 'Published.......run your custom shell script'"
              }]
      ]
}
```
- To run locally in dry run mode from repository
```
export GITHUB_TOKEN=<token> from https://github.com/settings/tokens (repo Permission)
semantic-release --dry-run
```

If you want to integrate semantic release with your CICD pipelines visit this [documentation](https://github.com/semantic-release/semantic-release/blob/master/docs/usage/ci-configuration.md).
I have enabled semantic release with Github actions for my maven project [factory-parent](https://github.com/BhuwanUpadhyay/factory-parent).
- [CI Action Workflows](https://github.com/BhuwanUpadhyay/factory-parent/blob/master/.github/workflows)
- [Release Artifacts](https://mvnrepository.com/artifact/io.github.bhuwanupadhyay/factory-parent)

The sample yaml file for Github action workflow to run semantic-release as below:

```yaml
name: Release

on: [ push ]

jobs:
  release:
    needs: build
    name: Semantic Release
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/master'
    steps:
      # check out repository code and setup node
      - uses: actions/checkout@v2
      - uses: actions/setup-node@v1
        with:
          node-version: "12.x"
      # install dependencies and run semantic-release
      - run: npm i -g semantic-release @semantic-release/{git,exec,changelog}
      - run: semantic-release
        env:
          GITHUB_TOKEN: <github_token>
```
