OpenContrail CI is now live. All commits to contrail-controller git repo will now have to go through CI. This process is very similar to [OpenStack](https://wiki.openstack.org/wiki/Gerrit_Workflow)'s. Pull requests are not longer used for contrail-controller git repo. Necessary .gitreview files are already in place in the repos that are administered through this CI infrastructure.

[Presentation Slides](https://github.com/Juniper/contrail-infra-config/blob/master/setup/OpenContrailCI.pptx)

[Juniper Internal Wiki](https://junipernetworks.sharepoint.com/teams/mvp/Contrail/_layouts/15/start.aspx#/Contrail%20Wiki/Contrail%20CI.aspx)

[CI Project Status](https://github.com/Juniper/contrail-infra-config/blob/master/contrail-ci-todo.txt)

In order to use the new system, you need to 

1. Login to [review](review.opencontrail.org) using your launchpad open id
2. Goto settings and 
    1. Select unique user name (STEP USER)
    2. Select (add if necessary) preferred email address to be same as the one used in your [github] (github.com/Junuper) and launch-pad
3. Add your ssh public keys

### Code review process
1. repo init, repo sync etc.
2. cd controller (e.g.)
3. git review -s (Only required, when you need to do 'git review' for the first time in this git repo)
       It asks for username. You must use the one setup in STEP USER step above (at review.opencontrail.org)
       If keys are correct, it setups a remote named gerrit in your git config
       git-hooks are setup to generate unique change-id with each commit
4. Create a branch e.g. git checkout -b bugfix github/master (tracking github/master at github.com/juniper/...)
5. Make your changes
6. git commit and provide commit message (This has to be done, after git review -s is successfully complete only)
    Closes-Bug: #1234567 -- use 'Closes-Bug' if the commit is intended to fully fix and close the bug being referenced.

    Partial-Bug: #1234567 -- use 'Partial-Bug' if the commit is only a artial fix and more work is needed.

    Related-Bug: #1234567 -- use 'Related-Bug' if the commit is merely elated to the referenced bug.

    Please add a bug id with the right keyword to your commit message on a separate line and gerrit will create a link to the correct bugid(s).

7. If necessary: apt-get -y install git-review (or yum -y install git-review)
     Please see [this](https://bugs.launchpad.net/git-review/+bug/1337701) if you get a pkg_resources.DistributionNotFound error
8. To backup your changes in your private repo (optional)
     Fork off [Juniper/contrail-controller] (github.com/Juniper/contrail-controller) into your private repo at github.com
     git remote-add <ur-private-repo>
     git push <ur-private-repo> bugfix
9. run "git review"

First time when you run "git review", it asks for the user name. Please use the same that you used in step USER above. But you should do "git review -s" first, which does the setup.

If works correctly, a review entry is created in [review server](review.opencontrail.org) Jenkins jobs are run to verify your changes. Jenkins master can be accessed at [jenkins](jenkins.opencontrail.org)

Some one should review and (optionally some one else) must approve the changes. If jenkins job verification also succeeds, the changes get automatically merged and pushed out to [github repo](github.com/Juniper/contrail-controller)

## Interdependent changes across different projects (git repos)
If you have changes spread across different git repos, then CI cannot handle it, as it does so with only one git repo at a time. In such cases, please follow this process.

* **Please make sure that all new, modified and deleted files are add/deleted to/from git before git commit**
* Commit your changes and do 'git review' for all the change sets necessary.
* Get your changes reviewed and approved
* Stand alone changes, which does not break (build and tests) automatically will get merged via the usual CI process
* Run single node sanity tests (fab run_sanity:ci_sanity) using your image in both centos and in ubuntu (External to Juniper folks can request assistance from ci-admin@opencontrail.org for this part)
* Email the test results URL and all review entry URLs to ci-admin@opencontrail.org with a request to merge the changes
* For controller project, one has to run and send "scons test" and (BUILD_ONLY=TRUE scons flaky-test) command output as well. (exit code must be 0 for both).
* If some dependent commits are in git-repos which are not in CI, corresponding pull request URLs must also be present (for admin to merge at the right time)
* CI Team can then do the needful to get the changes merged together.

IOW, we follow what CI is doing for all other independent commits, thus keeping the build sane.

## FAQ

1. Cannot git commit due to "missing change-id message"..

    All commits must happen after git-review is issued one time (which sets up git-review git-commit hooks) to generate change-ids for each commit. cheery-pick from older direct commits are not allowed. In such cases, the diff can be patched and committed again (e.g. git show <commit-id> | patch -p1; git commit -m "commit msg" .;)

2. Review entry is struck, what to do ?

    Some times, zuul, the queue manager loses track of a review entry. In that case, one should abandon and restore the change again to feed the review entry back into the pipeline. Fresh tests are triggered, upon completion of which, changes would get merged upstream (provided review and approvals are complete)

3. How to submit a patch to an already submitted entry ?

    Please see OpenStack documentation. In short, checkout the previous entry changes (git review --download <review-entry-id>, make changes, git commit --amend, and git review again).  If the sandbox and branch where the original changes were made is still available, the first step (i.e. git review --download) can be skipped. Note that git-review fails if the change has been abandoned.

4. How long does it take for the tests to complete ?

    At the moment, it takes 4 to 5 hours. Work is in progress to speed this up, while at the same time add additional tests that run in parallel.

5. How to flip a job so that jobs are restarted

    Either add a comment "recheck no bug" or "Abandon and Restore" review entry. 

**e.g. Steps to clone a git repo off your private fork ("rombie" in this e.g.) and submit changes to review for contrail-packaging project to R1.10 branch.**

git clone git@github.com:rombie/contrail-controller.git contrail-controller            
cd contrail-controller                                                                    
git remote add github git@github.com:Juniper/contrail-controller                         
git fetch github
git checkout -b R1.10 github/R1.10
git review -s
   <Use the same user name that is selected in review.opencontrail.org>
<make your changes>
git commit -m "msg" .
git push rombie R1.10
  Optionally (to backup), push your changes to your private repo
git review      
  Submit changes to review.opencontrail.org                           

For all other Questions, please email to [ci-admin](mailto:ci-admin@opencontrail.org)

Thank You.

OpenContrail-CI-Team