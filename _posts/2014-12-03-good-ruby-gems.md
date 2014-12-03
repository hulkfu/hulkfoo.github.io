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

# [mechanize](https://github.com/sparklemotion/mechanize)
The Mechanize library is used for automating interaction with websites. Mechanize automatically stores and sends cookies, follows redirects, and can follow links and submit forms. Form fields can be populated and submitted. Mechanize also keeps track of the sites that you have visited as a history.

flickr上传照片：

```ruby
require 'rubygems'
require 'mechanize'

agent = Mechanize.new

# Get the flickr sign in page
page  = agent.get 'http://flickr.com/signin/flickr/'

# Fill out the login form
form          = page.form_with :name => 'flickrloginform'
form.email    = ARGV[0]
form.password = ARGV[1]
form.submit

# Go to the upload page
page  = page.link_with(:text => 'Upload').click

# Fill out the form
form  = page.forms.action('/photos_upload_process.gne').first
form.file_uploads.name('file1').first.file_name = ARGV[2]
form.submit
```

google搜索：

```ruby
require 'mechanize'
require 'logger'

agent = Mechanize.new
agent.log = Logger.new "mech.log"
agent.user_agent_alias = 'Mac Safari'

page = agent.get "http://www.google.com/"

search_form = page.form_with :name => "f"
search_form.field_with(:name => "q").value = "Hello"
search_results = agent.submit search_form

puts search_results.body
```

# [listen](https://github.com/guard/listen)

用来监听本地文件的变化。还能将事件通过TCP协议发送。

```ruby
listener = Listen.to('dir/to/listen', 'dir/to/listen2') do |modified, added, removed|
  puts "modified absolute path: #{modified}"
  puts "added absolute path: #{added}"
  puts "removed absolute path: #{removed}"
end
listener.start # not blocking
sleep
```
