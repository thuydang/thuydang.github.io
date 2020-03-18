---
layout: post
title: Howto copy a single folder from a github branch
categories: [blog, howto]
tags: [trick, github]
comments: true
#series: "Cloud lab with OpenStack"
---

Sometime it is required to get a latest patch from an open source project because it may take year(s) for the maintainers to merge the pull request. Hää?.. Show me howto just get a single folder from a github branch with **svn**. Hää?... Yes, SVN.

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
    svn checkout https://github.com/softagram/ansible/branches/pull-48870/lib/ansible/modules/net_tools

That's it.

----
****
