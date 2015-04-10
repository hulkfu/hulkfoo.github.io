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

# [foreman](https://github.com/ddollar/foreman)

通过Procfile配置文件来管理多进程的应用。

一个Rails应用的Procfile文件如下：

```
web:    bundle exec thin start -p $PORT
worker: bundle exec rake resque:work QUEUE=*
clock:  bundle exec rake resque:scheduler
```

在应用根目录执行**foreman start**即可同时启动上面的进程，它们以不同的颜色显示。

## 生产环境
foreman比较适合用在开发环境，然后它可以将配置导出到[upstart](http://upstart.ubuntu.com/)或标准Unix init来使用。

### Exporting to upstart

```
$ foreman export upstart /etc/init
[foreman export] writing: /etc/init/testapp.conf
[foreman export] writing: /etc/init/testapp-web.conf
[foreman export] writing: /etc/init/testapp-web-1.conf
[foreman export] writing: /etc/init/testapp-worker.conf
[foreman export] writing: /etc/init/testapp-worker-1.conf
[foreman export] writing: /etc/init/testapp-clock.conf
[foreman export] writing: /etc/init/testapp-clock-1.conf
```

导出后，就可以像如下使用了：

```
$ start testapp
$ stop testapp-clock
$ restart testapp-worker-1
```

### Exporting to init

```
$ foreman export inittab
# ----- foreman testapp processes -----
TE01:4:respawn:/bin/su - testapp -c 'PORT=5000 bundle exec thin start -p $PORT >> /var/log/testapp/web-1.log 2>&1'
TE02:4:respawn:/bin/su - testapp -c 'PORT=5100 bundle exec rake resque:work QUEUE=* >> /var/log/testapp/worker-1.log 2>&1'
TE03:4:respawn:/bin/su - testapp -c 'PORT=5200 bundle exec rake resque:scheduler >> /var/log/testapp/clock-1.log 2>&1'
# ----- end foreman testapp processes -----
```

## 并发
foreman支持每一个进程样本执行不只一个进程。

```
# run 1 of each process type, and 2 workers
$ foreman start -c worker=2

# do not run a clock process
$ foreman start -c clock=0
```

## 参考
* http://blog.daviddollar.org/2011/05/06/introducing-foreman.html

