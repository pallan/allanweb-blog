---
layout: post
title: "Chiliproject to Redmine Converter"
date: 2013-09-22 20:05
comments: false 
categories: [redmine, chiliproject, ruby]
---

My company has been using Redmine as a basic project management tool for
many years. A couple of years ago Redmine was forked by many of the
developers to a parallel project known as Chiliproject. At the time we
studied the roadmaps and goals of each fork and decided to move to
Chiliproject. As the two projects has just separated the transition was
fairly easy.

Fast forward two years, Redmine has a thriving community, has upgraded
to Rails 3, releases regular updates and has a plethora of plugins. 
Chiliproject as floundered in comparison. The only
updates that have been released in the past year have been to fix
security issues in Rails or Ruby.

It was because of this we made the fateful decision to switch back to 
Redmine. Unfortunately, the projects are much farther apart in terms of
the codebase and database structure.  Add to this, Chiliproject has truly 
engrained itself in our company. We needed a way to convert back to Redmine
without impacting the users.

After some research we came up with <a href="https://gist.github.com/pallan/6663018">this script</a> to do the job. It has fairly detailed comments, but if you notice any problems or have any comments feel free to add them to the gist.
