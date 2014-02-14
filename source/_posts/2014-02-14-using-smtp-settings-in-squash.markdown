---
layout: post
title: "Using SMTP Settings in Squash"
date: 2014-02-14 11:21
comments: true
categories: [ruby, squash, ActionMailer] 
---

Recently, we started using <a href="https://github.com/SquareSquash/web">Squash</a> 
to monitor exceptions from our company's internal applications.

We soon discovered, however, that we were not receiving emails from
Squash to notify us of errors. It turns out that by default Squash's
email setting use a localhost email configuration. For security reasons
our servers do not have these services enabled on servers that do not
require it. Therefore, we had to update Squash to send via our internal
Exchange system. This is how we did it.

### Add additional Gem requirements

We needed to use NTLM authentication so we added `ruby-ntlm` to the
Gemfile and tell it to only require the SMTP settings.

```
gem 'ruby-ntlm', require: 'ntlm/smtp'
```

### ActionMailer Settings

Squash uses Configoro to manage configurations which is not complicated to
use but we needed a quick fix to get things operating as we needed. As
such we added a new initializer with our settings:

```ruby
# config/initializers/mail.rb
 
ActionMailer::Base.smtp_settings = {
  :address  => "mail.host",
  :port  => 25,
  :domain  => 'example.com',
  :user_name => 'USER',
  :password => 'PASS',
  :authentication => :ntlm,
  :enable_starttls_auto => true
}
```

A quick app restart and email was back in business.

We have forked Squash and are working on a 'proper' solution for this
that we will submit back to the project as a Pull Request. I will try to
update this post when that patch is available and/or merged in.


