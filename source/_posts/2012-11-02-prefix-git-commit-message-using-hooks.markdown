---
layout: post
title: "Prefix Git Commit Message using Hooks"
date: 2012-11-02 10:01
comments: true
categories: [git, ruby]
---

My team uses <a href="http://chiliproject.org/">Chiliproject</a> to manage our user stories. We have a team agreement to ensure that each commit message into our git repository is prefixed with the issue ID that the commit is related to. In addition to this we have a branch naming convention that also identifies the issue that branch is supposed to address.  For example, a branch name could be, "rm-3454-customer-defect" and (hopefully) all commit messages will be prefixed with "RM #3454". Chiliproject has some nice little features with linking and styling when the "#4233" pattern is found. However, it can be fickle. If you forget to put in a space, for example, "RM#3432" none of it will work.

Making sure this formatting is correct can be a real pain in the a**.  Therefore, I took it upon myself to simplify things. The script below uses our branch naming convention to automatically prefix any commit message using the git "commit-msg" hook. You can see the code <a href="https://gist.github.com/pallan/4001241">is this gist</a>.

Put this code in the ".git/hooks/commit-msg" file at the root of your repository, make it executable and you are ready to rumble (of course, this will only work if using ruby).


