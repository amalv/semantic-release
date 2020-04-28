# Semantic Release example using Next.js

This is a really simple project that shows the usage of Semantic Release with Next.js

You can find the original article [here](https://dev.to/amalv/how-to-setup-semantic-release-for-a-react-app-or-a-next-js-app-25c1)

## Overview

In this guide you'll learn how to setup [Semantic Release](https://github.com/semantic-release/semantic-release) for a [React](https://reactjs.org/) app or a [Next.js](https://nextjs.org/) app.

This will allow you to follow a workflow that performs fully automated releases for Github and enforces the [Semantic Versioning](https://semver.org/) specification based on your commit messages.

Here is an example from the [official documentation](https://github.com/semantic-release/semantic-release/blob/master/README.md) of the release type that will be done based on commit messages:

| Commit message                                                                                                                                                                                   | Release type               |
|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|----------------------------|
| `fix(pencil): stop graphite breaking when too much pressure applied`                                                                                                                             | Patch Release              |
| `feat(pencil): add 'graphiteWidth' option`                                                                                                                                                       | ~~Minor~~ Feature Release  |
| `perf(pencil): remove graphiteWidth option`<br><br>`BREAKING CHANGE: The graphiteWidth option has been removed.`<br>`The default graphite width of 10mm is always used for performance reasons.` | ~~Major~~ Breaking Release |


Tools such as [commitizen](https://github.com/commitizen/cz-cli) or [commitlint](https://github.com/conventional-changelog/commitlint) can be used to enforce valid commit messages.

You can use the [commitizen extension](https://marketplace.visualstudio.com/items?itemName=KnisterPeter.vscode-commitizen) to add commitizen support to Visual Studio Code.

## Basic setup

First, create a Next.js app using [Create Next App](https://create-next-app.js.org/):

```bash
npx create-next-app semantic-release --example with-typescript --use-npm
```

Or if you prefer to use only React, use [Create React App](https://github.com/facebook/create-react-app) and run:

```bash
npx create-react-app semantic-release --template typescript --use-npm
```

## Create a Github repository

https://github.com/new

In this example I called it: `semantic-release`

## Link local repository to Github repository

```bash
git remote add origin git@github.com:<username>/<repository-name>.git
git push -u origin master
```

## Github token

A Github token must be created in order for Semantic Release to be able to publish a new release to the Github repository.

You can read [here](https://help.github.com/en/github/authenticating-to-github/creating-a-personal-access-token-for-the-command-line) how to create a token for Github. You need to give the token repo scope permissions.

Once you have the token, you have to save it in your repository secrets config:

```bash
https://github.com/<username>/<repositoryname>/settings/secrets
```

Use `GH_TOKEN` as the secret name.

## Install Semantic Release Git and Changelog plugins

```bash
npm install --save-dev @semantic-release/git @semantic-release/changelog
```

These plugins are necessary in order to create a Changelog and publish the new release in Github.

## Add Semantic Release config to package.json

Add the following config to your package.json:

```bash
  "private": true,
  "plugins": [
    "@semantic-release/commit-analyzer",
    "@semantic-release/release-notes-generator",
    "@semantic-release/changelog",
    "@semantic-release/github",
    "@semantic-release/npm",
    "@semantic-release/git"
  ],
  "release": {
    "prepare": [
      "@semantic-release/changelog",
      "@semantic-release/npm",
      {
        "path": "@semantic-release/git",
        "assets": [
          "package.json",
          "package-lock.json",
          "CHANGELOG.md"
        ],
        "message": "chore(release): ${nextRelease.version} [skip ci]\n\n${nextRelease.notes}"
      }
    ]
  }
```

When you set the `private` property to true, it skips the publication to the NPM repository, which in this case we don't want to do.

## Setup Github Actions

Github Actions will be used to create new releases of your app.

You must store workflows in the `.github/workflows` directory in the root of your repository. Once you created the directories, add a `main.yml` file inside with the following config:

```bash
name: Semantic release

on:
  push:
    branches:
      - master
jobs:
  publish:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - name: Setup Node.js
        uses: actions/setup-node@v1
        with:
          node-version: 12
      - name: Install dependencies
        run: npm install 
      - name: Build app
        run: npm run build 
      - name: Semantic release
        env:
          GITHUB_TOKEN: ${{ secrets.GH_TOKEN }}
        run: npx semantic-release
```

## Commit and push changes

Use the following commands:

```bash
git commit -m "feat: Add Semantic Release and Github Actions"
git push
```

## Conclusion

Congratulations!

If you followed these steps, you should now have triggered Github Actions:

![Github Actions](https://dev-to-uploads.s3.amazonaws.com/i/zutm6lg2cl26yg2b961k.png)

Also, if you check the release tab in your repository, you'll also see your first published release:

![Github Release](https://dev-to-uploads.s3.amazonaws.com/i/b7lno809y3h66shlq9ob.png)

And finally, a Changelog file should have been automatically generated and published:

![Changelog](https://dev-to-uploads.s3.amazonaws.com/i/iw87456b3mw89i3glk7z.png)