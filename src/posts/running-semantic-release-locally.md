---
title: Running Semantic Release Locally
date: 2016-07-29 10:21:28
tags:
  - semantic-release
  - ci
  - meta-development
  - github
---

[Semantic Release](https://github.com/semantic-release/semantic-release) is a great tool for managing release versions, and auto generating a [changelog](https://en.wikipedia.org/wiki/Changelog)/ or [github release notes](https://github.com/blog/1547-release-your-software) for you software based on your commit messages.

It helps enforce consistent style across a project and forces developers to think about the code their committing and the changes that it will have on the larger application.

By default [Semantic Release](https://github.com/semantic-release/semantic-release) expects to be run on Travis-ci (or some other [continuous integration](https://en.wikipedia.org/wiki/Continuous_integration) tool. I often find however that for small projects (like [Material Calculator](https://materialcalculator.com)) that setting this up is overkill. By default [Semantic Release](https://github.com/semantic-release/semantic-release) only does a dry run locally. Here's how to trick it into doing the real thing:

Create a `.env` file in your project's root with the following variables filled in:

```bash
export CI=1
export TRAVIS=true
export TRAVIS_BRANCH=master
export GH_TOKEN={{ YOUR PRIVATE KEY }}
export NPM_TOKEN={{ YOUR PRIVATE KEY }}
```

You'll also want to add this to your `.gitignore` as these private keys shouldn't be public.
