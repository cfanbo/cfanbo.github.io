---
title: git中的Reversing Pushed Changes
author: admin
type: post
date: 2012-04-07T06:44:37+00:00
url: /archives/12697
IM_contentdowned:
 - 1
categories:
 - 程序开发
tags:
 - git

---
### Reversing Pushed Changes

Sometimes you or somebody else might have pushed changes accidentally to the remove repository. To get rid of them, first get a log or history of the push commits:

```
$ git log
```

Then, use `git reset` to push back to a particular come it, identified by its SHA1 sequence from the log. For example:

```
$ git reset --hard 6bb3dc30bc0c8fc36421474cf9376d658ee643aa
```

Sometimes just the first few letters and numbers of the sequence, such as `6bb3dc` would do.

After you’ve done the reset, you need to push it back to the server. However, if you just pushed your branch, you will get an error message:

```
$ git push git@gitorious.org:opentaps/opentaps.git master
To git@gitorious.org:opentaps/opentaps.git
 ! [rejected]        master -> master (non-fast-forward)
error: failed to push some refs to 'git@gitorious.org:opentaps/opentaps.git'
To prevent you from losing history, non-fast-forward updates were rejected
Merge the remote changes before pushing again.  See the 'Note about
fast-forwards' section of 'git push --help' for details.
```

To really push it, you would need to add a `+` before your branch name:

```
$ git push git@gitorious.org:opentaps/opentaps.git +master
Total 0 (delta 0), reused 0 (delta 0)
=> Syncing Gitorious... [OK]
To git@gitorious.org:opentaps/opentaps.git
 + 6398f5f...6bb3dc3 master -> master (forced update)
```