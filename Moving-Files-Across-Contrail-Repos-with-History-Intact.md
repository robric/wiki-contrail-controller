# [How To](https://stosb.com/blog/retaining-history-when-moving-files-across-repositories-in-git/)
The procedure to move files across repos is outlined at [stosb](https://stosb.com/blog/retaining-history-when-moving-files-across-repositories-in-git/). Fundamentally, any file or directory move (across repos) must be viewed as a sequence of two steps: repo split followed by repo merge (of one of the split-parts with the destination repo). A repo is split into two, and one of the split-parts that requires to be moved is merged with destination repo. In instances where entire source repo is merged with a destination repo, skip the split aspect and proceed to the merge step (However, most of the times it may not always be the case, one of the reasons is listed below). 

[A simple “git mv” does not preserve git-history](https://git.wiki.kernel.org/index.php/GitFaq#Why_does_Git_not_.22track.22_renames.3F), and requires in some instances ‘git log’ command be given a –follow option etc., and give an incorrect impression that "git mv" can preserve history. [A quote from Stackoverflow](https://stackoverflow.com/questions/2314652/is-it-possible-to-move-rename-files-in-git-and-maintain-their-history): It is really annoying that so many people have mindlessly repeated the statement that git automatically tracks moves. Git does no such thing. [By design(!) Git does not track moves at all](https://git.wiki.kernel.org/index.php/GitFaq#Why_does_Git_not_.22track.22_renames.3F).  

Whenever a repo is merged (referred to as source repo) with a destination repo, all the contents of the source repo are spit (or spewed) into the root directory of the destination repo. In most instances, it is not the desired directory structure. One cannot rely on ‘git mv’ to preserve history for the reasons listed above. As a result, one will have to split the source repo into two repos, a desired part and the other, and merge the desired part of the repo with destination repo. One may end up repeating this process (once per each directory that needs to be moved from the source repo) because of desired directory layout in the destination repo. Once split, the source repo is unusable (the set of git commands invoked to achieve the split make the repo unusable for further splits), hence mandates that a fresh repo be pulled for every repetition (that would merge the next directory, implied in the fresh repo pull is repetition of all the steps).  

Summary: Move of files across repo can be viewed as two steps, split and merge. One will have to repeat the split and merge steps per directory (per every top level directory under the repo root that requires move).

**Split a repo**
* Split an existing repo into two
* Part that you requires merge with a destination repo
* And the OTHER part

**Merge a repo**
* Pick the split-part of repo (from above) and merge with a destination repo

The example below walks through how controller/src/base is split into a repo ([a local repo](http://blog.osteele.com/2008/05/my-git-workflow/)) of its own and merged with src/contrail-common. 

**Split a repo: Split contrail-controller into a repo that contains only base:**
Currently, base is part of the contrail-controller repo. Let’s split the controller/src/base from contrail-controller and create a new (local) repo.
* cd controller
* git checkout –b master
* Run all the commands from the root directory of the source repo (in this case controller)
* create a file called script with following lines:

    ``#!/bin/bash``

    ``/bin/mkdir –p newroot/``

    ``/bin/mv src/base newroot/``

    ``true``

    ``(base is the directory you are moving here. Save the above file as controller/script)``

* In this instance the file:script is located in the directory controller (at the same level as the directory being moved)
* git filter-branch –f –prune-empty –tree-filter /build/username/mainline_build/controller/script HEAD
* git filer-branch –prune-empty –f subdirectory-filter newroot
* git remote –v
* git remote rm github (or whatever your remote, optional step)
* The above steps create a local repo that contains base (base: the directory you want to move)
* At this point the repo is split and you have the desired part of the repo, a new (local) repo, with ONLY base (the directory you want to move) at the root of the repo

**Merge a repo: Merge the new local repo (pruned controller with just base) with src/contrail-common:**
* cd src/controller-common (root directory of your destination repo)
* git checkout –b merging_base
* git remote add moving_base ../../controller (this is the patch to the root of your source repo)
* Moving_base is the name of your repo
* git remote –v will show (moving_base) after this command
* git fetch moving_base
* git merge moving_base/master (build, test, make necessary changes to scons, rules.py etc.)
* git push -u github (whatever is your remote) merging_base
* create a pull request on github and merge your changes


