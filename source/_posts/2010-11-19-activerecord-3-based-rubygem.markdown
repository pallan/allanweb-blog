---
layout: post
title: "ActiveRecord 3 based Rubygem"
date: 2010-11-19 21:12
comments: true
categories: [ruby, activerecord, rubygem] 
---

At our company we had to integrate with a legacy database system. To manage it we created a nice little gem to handle it. Basically the gem was just a collection of ActiveRecord models, nicely namespaced, so that we could include it in any application that made have needed access to that db. Code using the gem set it up as you would expect (in a Rails app we used a custom db connection in the database.yml):

```ruby
require 'legacy_database'
LegacyDatabase::Base.establish_connection(...)
```

This functioned great for many years and versions of ActiveRecord.

Jump forward to today, we have another separate database we need to connect to. Naturally, after the success of the previous gem we decided to create a new gem using the existing pattern, but this time we are going to use the new and fancy ActiveRecord 3 and its nice new Arel syntax. 

We started building the gem and immediately ran into a blocker.  Using the same code as above, as soon as the require was executed we get an ActiveRecord::ConnectionNotEstablished error. Nuts!

```ruby
require 'special_database' # &lt;= raises ActiveRecord::ConnectionNotEstablished
SpecialDatabase::Base.establish_connection(...)
```

The backtrace showed us that when the gem was being loaded when it hits the first scope call Arel attempts to connect to the database.  Why? I am not sure and I haven't had time to dig around to find out if it is even possible to fix. Its a classic chicken/egg situation. You can't initialize the connection until you load the gem and you can't load the gem without initializing the connection.

In our situation we aren't using the gem in concert with any other ActiveRecord connections so we are able to use ActiveRecord::Base to establish the connection.

```ruby
require 'activerecord'
ActiveRecord::Base.establish_connection(...)

require 'special_database'
SpecialDatabase::Foo.find(1)  # It work's
```

I really don't like it, but so far its our only option. Got a better solutions I'd be happy to hear it.
