OpenContrail CI is now live. All commits to contrail-controller git repo will now have to go through CI. This process is very similar to [OpenStack's](https://wiki.openstack.org/wiki/Gerrit_Workflow)'s. Pull requests are not longer used for contrail-controller git repo. Necessary .gitreview files are already in place in the repos that are administered through this CI infrastructure.

In order to use the new system, you need to 

1. Login to [review](review.opencontrail.org) using your launchpad open id
2. Goto settings and 
    1. Select unique user name (STEP USER)
    2. Select (add if necessary) preferred email address to be same as the one used in your [github] (github.com/Junuper) and launch-pad
3. Add your ssh public keys

## Code review process
1. Fork off [Juniper/contrail-controller] (github.com/Juniper/contrail-controller) into your private repo
2. Create a branch e.g. bugfix
3. Make your changes
4. Commit
5. If necessary: apt-get -y install git-review (or yum -y install git-review)
6. run "git review"

First time when you run "git review", it asks for the user name. Please use the same that you used in step USER above.

If works correctly, a review entry is created in [review server](review.opencontrail.org) Jenkins jobs are run to verify your changes. Jenkins master can be accessed at [jenkins](jenkins.opencontrail.org)

Some one should review and (optionally some one else) must approve the changes. If jenkins job verification also succeeds, the changes get automatically merged and pushed out to [github repo](github.com/Juniper/contrail-controller)


## FAQ

1. Cannot git commit due to "missing change-id message"..
    All commits must happen after git-review is issued one time (which sets up git-review git-commit hooks) to generate change-ids for each commit. cheery-pick from older direct commits are not allowed. In such cases, the diff can be patched and committed again (e.g. git show <commit-id> | patch -p1; git commit -m "commit msg" .;)

2. Review entry is struck, what to do ?
    Some times, zuul, the queue manager loses track of a review entry. In that case, one should abandon and restore the change again to feed the review entry back into the pipeline. Fresh tests are triggered, upon completion of which, changes would get merged upstream (provided review and approvals are complete)

3. How to submit a patch to an already submitted entry ?
    Please see OpenStack documentation. In short, checkout the previous entry changes (git review --download <review-entry-id>, make changes, git commit --amend, and git review again)

4. How long does it take for the tests to complete ?
    At the moment, it takes 4 to 5 hours. Work is in progress to speed this up, while at the same time add additional tests that run in parallel.

For all other Questions, please email to [ci-team](mailto:ci-team@opencontrail.org)

Thank You.
OpenContrail-CI-Team
