---
title: Handling bookmarks in Jujutsu
layout: post
date: 2025-07-05T22:28:00-0700
---

Many newcomers to [Jujutsu] (jj) have trouble with how it handles bookmarks.

Bookmarks in jj are essentially Git branches. In fact, when you push a
bookmark, it becomes a branch on the remote.

The essential difference between Jujutsu bookmarks and Git branches is that
Jujutsu bookmarks do not advance automatically as you create commits. To move
them from one change to another, you must use `jj bookmark move` or `jj
bookmark set`.

[Jujutsu]: https://jj-vcs.github.io/jj/

## Why do people expect bookmarks to advance, anyway?

The workflow many developers use to contribute work is:

- Create a new feature branch
- Create a commit with your work
- Push
- Improve your work and address feedback by adding more commits to the branch

A feature branch may start with a small number of commits:

```shell
○ yyy  [feature-b] Add feature B
│
○ zzz  Refactor component A
│
○ [main]
```

But after addressing feedback, the commit history starts to look like this:

```shell
○ vvv  [feature-b] Rename new function
│
○ www  oops, fix test
│
○ xxx  Tweak changes to component A
│
○ yyy  Add feature B
│
○ zzz  Refactor component A
│
○ [main]
```

## Simplest solution: Amend existing revisions

Jujutsu makes it very easy to amend revisions. So, instead of appending new
revisions to your feature branch, amend the revisions in place.

If the two revisions in your branch are _Refactor component A_ and _Add feature
B_, most feedback given during code review should be amended into one of these
two commits.

Suppose you realize you should refactor component C before implementing your
feature. You can insert a _new_ revision and implement it there:

```shell
$ jj new -A zzz -B yyy -m 'Refactor component C'
```

This command inserts a new revision between `zzz` and `yyy`.

### Effect on others

Won't this mess up others? Many have learned a rule similar to: _Don't rewrite
commits that you've pushed_. If you do, others will have to [recover from an
upstream rebase][recover-rebase].

But, will they? They'd only need to do that if they base their work off of
yours. Does your team base work off of each other's branches? If not, then
there's no worry. And even if they do, recovering is not _that_ difficult:

```shell
$ git rebase --onto feature-b feature-b@{1}
```

## Another option: `jj tug`

Sometimes, amend commits isn't feasible. For example, some teams simply have a
rule not to rebase feature branches that have been pushed.

Many people use an alias called [`jj tug`] to handle this. This alias finds the
closest bookmark in history and moves it to either `@` or `@-`. There are a few
versions of the alias, but an example is:

```toml
[revset-aliases]
'closest_bookmark(to)' = 'heads(::to & bookmarks())'
'closest_pushable(to)' = 'heads(::to & ~description(exact:"") & (~empty() | merges()))'

[aliases]
tug = ["bookmark", "move", "--from", "closest_bookmark(@)", "--to", "closest_pushable(@)"]
```

[`jj tug`]: https://github.com/jj-vcs/jj/discussions/5568

[recover-rebase]: https://git-scm.com/docs/git-rebase#_recovering_from_upstream_rebase

