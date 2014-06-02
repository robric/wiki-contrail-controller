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

For any Qs, please email to [ci-team](mailto:ci-team@opencontrail.org)

Thanks,
OpenContrail-CI-Team
