---
layout: post
title: 好用的ruby gem
---

收集好用的ruby gem。

# [mail](https://github.com/mikel/mail)
发邮件十分方便。

```ruby
require 'rubygems'
require 'mail'
begin
  require 'ruby-debug'
rescue LoadError
  STDERR.puts "Continuing without ruby-debug"
end

smtp = { :address => 'mail.fiendz.org', :port => 587, :domain => 'fiendz.org', :user_name => 'test@fiendz.org', :password => 'foobar', :enable_starttls_auto => true, :openssl_verify_mode => 'none' }
Mail.defaults { delivery_method :smtp, smtp }
mail = Mail.new do
  from 'test@fiendz.org'
  to 'donald.ball@gmail.com'
  subject 'subject'
  body 'body'
end

# add attachments
mail.attachments['test.png'] = File.open('path/to/test.png', 'rb', &:read)

mail.deliver!
```