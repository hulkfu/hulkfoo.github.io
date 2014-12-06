---
layout: post
title: Capistrano 3
---

代码很清晰，分为配置和任务。在lib中，除了tasks定义的任务外，dsl和templete就是为configuration服务的。

在模板中，通过dsl来设置服务器的信息。

set设置，fetch获得（不知为什么不用get）。

```ruby
def set(key, value)
  pval = @properties[key]
  if pval.is_a? Hash and value.is_a? Hash
    pval.merge!(value)
  elsif pval.is_a? Set and value.is_a? Set
    pval.merge(value)
  elsif pval.is_a? Array and value.is_a? Array
    pval.concat value
  else
    @properties[key] = value
  end
end
```

看set方法，如果是set，hash或array就合并，否则就赋值，所以可以多次使用。

设置好后，cap production deploy执行，就可以将代码部署到服务器上了。这就是task干的。

当然如果没有设置部署后要执行的脚本，比如启动unicron，网站是不能正常访问的。最简单的莫过于直接执行

```
bundle exec rails server -p 80 -b THE_SERVER_IP -d -e production
```

在deploy.rb里这么定义：

```ruby
namespace :deploy do
  set :server_pid, "#{current_path}/tmp/pids/server.pid"

  desc 'Start application'
   task :start do
    on roles(:app), in: :sequence, wait: 5 do
      within current_path do
        if test("[ -e #{fetch(:server_pid)} ] && kill -0 #{pid}")
          info "application is running..."
        else
          with rails_env: "production" do
            execute :bundle, :exec, :rails, :server, "-p PORT -b THE_SERVER_IP -d"
          end
        end
      end
    end
  end
end

def pid
  "`cat #{fetch(:server_pid)}`"
end
```

注意：上面使用execute，如果使用的是rvm，直接把命令放到一起执行不可，非要如上用参数形式。因为后者其实是执行的

~/.rvm/bin/rvm default do bundle exec rails server

对，多了**~/.rvm/bin/rvm default do**.

综上，只要理解了一个工具的思想，那么用起来就会有理有据，得心应手，而不会摸不着头脑，每一步都有每一步的位置。


# 参考
https://github.com/jesson/capistrano3-nginx_unicorn
http://stackoverflow.com/questions/6018591/capistrano-deploy-with-thin-servers
