---
layout: post
title: Howto copy a single folder from a github branch and apply pull request
categories: [blog, howto]
tags: [trick, github, patch]
comments: true
#series: "Cloud lab with OpenStack"
---

Sometime it is required to get a latest patch from an open source project because it may take year(s) for the maintainers to merge the pull request. Hää?.. Show me howto just get a single folder from a github branch with **svn**. Hää?... Yes, SVN.

## Download code from pull request
Example. We want to get a patch for nmcli module of ansible by copying it the library folder in our ansible playbooks folder so ansible can pick it from there instead of using the _old_ module without the patch. The github url:

    https://github.com/softagram/ansible/tree/pull-48870/lib/ansible/modules/net_tools

Luckily github can also serve svn api by converting the git repo to svn one on the fly!. Try:

    svn ls https://github.com/softagram/ansible/
    branches/
    tags/
    trunk/

We want the pull-48870 which is in 'branches' folder by simply replace 'tree' in the original url with 'branches' and keep the rest of the url and checkout the folder under ansible 'library' folder:

    svn checkout https://github.com/softagram/ansible/branches/pull-48870/lib/ansible/modules/net_tools

Use export if we don't want to track the branch (it's easy to change the code and forget we are pushing to the original repo while thinking we are pushing the parent folder to our repo!):

    svn export https://github.com/softagram/ansible/branches/pull-48870/lib/ansible/modules/net_tools

That's it. Read on for more tips.

## Download devel branch and apply pull request as patch

The trick above works for single patch. If multiple PRs are needed, they must be applied to the same branch (devel). 


The steps are i) to download devel code based then 2) to get patches from PR and apply them as shown following.

### Dowload a folder from devel branch
As shown above but with ansible devel branch:

    cd library/
    svn export https://github.com/ansible/ansible/trunk/lib/ansible/modules/net_tools

### Get the pull request 

Let say we want the fix by the pull request [#59234](https://github.com/ansible/ansible/pull/59234). 

First we download the PR as mail-patch. Every pull-request on Github can be downloaded as a mail-patch, which has e-mail metadata, by appending ".patch" to the PR URL.

    cd library/net_tools/
    wget https://github.com/ansible/ansible/pull/59234.patch

After downloaded, apply the patch using:
    
    git am 59234.patch

## Merge PR

Unfortunately the patches are often applied to different fork of the origin code base. So we have to checkout and merge. As an example we will merge an ansible PR request.

Get latest ansible (devel) in `/opt`:

    git clone https://github.com/ansible/ansible.git

Apply and merge latest pull requests required. We will fix nmcli module for creating bond slaves as an example. 

Fixed by pull request [#59234](https://github.com/ansible/ansible/pull/59234). 

The patch is applied for previous fork of ansible, which is hundred of commits behind. So we will fetch it in a separate branch and merge with latest code base.

Fetch the PR to a branch `pr59234`:

    git fetch origin refs/pull/59234/head:pr59234

Now we can diff against the pull-requested commits, log them, cherry-pick them, or just merge them outright.

    git checkout devel
    git merge pr59234


That's all.

See 
* <https://coderwall.com/p/_8dxjw/merging-a-pull-request-locally>


----
****
