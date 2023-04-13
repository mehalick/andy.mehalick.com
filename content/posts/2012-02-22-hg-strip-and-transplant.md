---
layout: post
title: "Hg Strip and Transplant"
date: 2012-02-22
post_id: 3
---

It’s rare that I reach to my Mercurial toolbelt to wield these two Hg commands but it happens on occasion and that’s enough to warrant a quick summary of each. Both are distributed with Mercurial but not enabled by default (more on that below).

## Strip

Strip is a simple command to delete one or more changesets; typically used to remove commits that may have mistakes or to remove a set of experimental commits that you want to back out of. You’ll definitely want to avoid stripping changesets already pushed out to others, it’s much better to only strip changesets that live locally.

So let’s say you start up a new feature branch and make a couple of commits only to realize you’ve made some really bad design choices. There are many reasons why you should want to preserve all commits, good or bad; but let’s ignore those for now and proceed with our strip anyways. In our example let’s assume there are problems with changeset 11 (and subsequently 12):

![](https://andy.azureedge.net/blog/strip-commits-before-636217950924844916.png)

Simply execute **hg strip** with the revision number to want to remove; all higher revisions in the branch will be removed and revisions numbers in other branches will be reset:

```shell
hg strip 11
```

You’ll be left with the branch structure below, notice how changeset 13 was renumbered down to 11:

![](https://andy.azureedge.net/blog/strip-commits-after-636217950916380477.png)

Stripped changesets will be stored in “.hg/strip-bundle” and can be restored if necessary; the existing revision numbers will be preserved and your restored commits will get new revision numbers from there. You’ll need to find the bundle file name and execute something like:

```shell
hg unbundle .hg/strip-backup/0e5dd67a89f4-backup.hg
```

![](https://andy.azureedge.net/blog/strip-commits-unbundled-636217950931211165.png)

## Transplant

Transplant is a command that copies one or more changesets from one branch to another. I use this command occasionally when I’ve made very simple changes to a development branch that I later decide need to be pushed out to say a release or default branch. Assume you want to copy changesets 10 and 11 from the develop branch to the release branch:

![](https://andy.azureedge.net/blog/transplant-before-636217950944779307.png)

Start by updating your local working directory to the branch that will receive the changesets (release) and then execute transplant by passing in the name of the source branch (develop) and one or more revision numbers:

```shell
hg transplant –b develop 10 11
```

![](https://andy.azureedge.net/blog/transplant-after-636217950938090795.png)

## Enabling Strip and Transplant

Both commands are part of extensions that are distributed with Mercurial but will need to be enabled. Strip is included with [Mercurial Queues Extensions](http://mercurial.selenic.com/wiki/MqExtension) and transplant in its own [Transplant Extension](http://mercurial.selenic.com/wiki/TransplantExtension). Simply add them to your Mercurial.ini file:

![](https://andy.azureedge.net/blog/2012-02-22-10-24-46-pm-636217950899937804.png)