# Historic record of Archlinux Ceph

**DO NOT USE THIS FOR BUILDS**

This git branch contains the commit history of the original repo,
and the work done before this package was dropped from the Archlinux
community repos.

This branch is primarily for looking up how this repo came to be in
the state that it is at the root of `master`. Notably, `master` was
actually imported from fdb99ba, not the most recent 15.2.17-1 update.

It was captured from https://github.com/archlinux/svntogit-community/tree/packages/ceph
using the following commands:

```zsh
temp=$(mktemp -d)

git clone https://github.com/bazaah/aur-ceph
git clone \
  --single-branch \
  --branch packages/ceph \
  https://github.com/archlinux/svntogit-community community.ceph

( cd community.ceph ; \
  git format-patch -1 --output $temp/0000-genesis.patch 9eab698 ; \
  git format-patch -o $temp 9eab698 trunk )

( cd aur-ceph ; \
  git switch --orphan historic/archlinux.community.ceph ; \
  git am --committer-date-is-author-date -p2 $temp/* )
```
