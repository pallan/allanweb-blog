---
layout: post
title: 'Brew, git 1.8.0 & "-bash: __git_ps1: command not found"'
date: 2012-11-19 08:06
comments: true
categories: [git, homebrew]
---

I updated my machine this past weekend. This included everything from <a href="http://www.apple.com/osx/">Mountain Lion</a> to <a href="http://mxcl.github.com/homebrew/">Homebrew</a> (including all formulas).

As part of this update I moved from git 1.7.2.x to 1.8.0. After I did so and launched a new terminal I started seeing this fun error

```
-bash: __git_ps1: command not found
```

I checked the git 1.8.0 release notes but could not see any mention of a change to the prompt scripts.  <a href="http://www.spinics.net/lists/git/msg191753.html">After much research</a> (and gnashing of teeth) I discovered that the git shell commands that were responsible for making that nice prompt were moved from contrib/completion/git-completion.bash into a separate file contrib/completion/git-prompt.sh

To fix it I added the following line to my ~/.bash_profile above where I defined my prompt

```
source /usr/local/etc/bash_completion.d/git-prompt.sh
```

Where /usr/local/etc/bash_completion.d is where brew puts the completion scripts it installs.

I am no bash guru so I will not advocate that this is the best way to fix this problem.If you are a bash guru and know a better way please comment so we can all learn something.
