OpenContrail CI is now live. All commits to contrail-controller git repo will now have to go through CI. This process is very similar to [OpenStack's](https://wiki.openstack.org/wiki/Gerrit_Workflow)'s. Pull requests are not longer used for contrail-controller git repo. Necessary .gitreview files are already in place in the repos that are administered through this CI infrastructure.

In order to use the new system, you need to 

o Login to [review](review.opencontrail.org) using your launchpad open id
o Goto settings and 
    - Select unique user name (STEP USER)
    - Select (add if necessary) preferred email address to be same as the
      one used in your [github](github.com/Junuper) and launch-pad
    - Add your ssh public keys

Code review process
===================
o Fork off [Juniper/contrail-controller] (github.com/Juniper/contrail-controller) into your private repo
o Create a branch e.g. bugfix
o Make your changes
o Commit
o If necessary: apt-get -y install git-review
o run "git review"

First time when you run "git review", it asks for the user name. Please use the same that you used in step USER above.

If works correctly, a review entry is created in [review server](review.opencontrail.org) Jenkins jobs are run to verify your changes. Jenkins master can be accessed at [jenkins](jenkins.opencontrail.org)

Some one should review and (optionally some one else) must approve the changes. If jenkins job verification also succeeds, the changes get automatically merged and pushed out to [github repo](github.com/Juniper/contrail-controller)

For any Qs, please email to [ci-team](mailto:ci-team@opencontrail.org)

Thanks,
OpenContrail-CI-Team
