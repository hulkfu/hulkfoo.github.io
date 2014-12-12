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

综上，只要理解了一个工具的思想，那么用起来就会有理有据，得心应手，而不会摸不着头脑，每一步都有每一步的位置。

# 辅助gem

## capistrano/rvm

它会帮助设置server上的rvm环境，甚至不用配置，默认就很好。

在执行前，会先rvm:hook。

## capistrano/bundler

自动添加"bundle exec"

## [capistrano/sshkit](https://github.com/capistrano/sshkit)

在capistrano中，在远程服务器上执行命令。

它可以定义在rakefile里单独使用：

```
require 'sshkit'
require 'sshkit/dsl'

SSHKit::Backend::Netssh.configure do |ssh|
  ssh.ssh_options = {
      user: 'deploy',
      auth_methods: ['publickey']
  }
end

desc 'Who am I on the server'
task :who do
  on '1.example.com' do
    as :deploy do
      puts capture(:whoami)
    end
  end

end

desc 'Some task'
task :some_task do
  on %w{1.example.com 2.example.com}, in: :sequence, wait: 5 do
    within "/opt/sites/example.com" do
      as :deploy  do
        with rails_env: :production do
          rake   "assets:precompile"
          runner "S3::Sync.notify"
          execute "node", "socket_server.js"
        end
      end
    end
  end
end
```

在Rails中，只是把配置何任务等分到了不同的文件里。

### DSL

on()，指定要执行命令的server。并且可以设定执行的方式：parallel、sequence、groups。

within()，设定一个目录来执行命令。

as()，执行命令的用户名。

with()，用来设置命令执行的环境等。

### ssh方法

它们定义在backends/netssh.rb里。

run()，执行之前初始化Local或Netssh定义在其block里的代码。在dsl.rb里已经自动执行了。

test()，加入测试参数，然后execute。

capture()，捕获命令执行的结果为string。

execute()，执行一个shell命令。

upload!()，使用scp上传。

download!()，使用scp下载。

### 注意

上面使用execute，如果使用的是rvm，直接把命令放到一起执行不可，非要如上用参数形式。因为后者其实是执行的

~/.rvm/bin/rvm default do bundle exec rails server

对，多了**~/.rvm/bin/rvm default do**.

原因在其[The Command Map](https://github.com/capistrano/sshkit#the-command-map)上有解释：当命令中有空格或是新行时，不能够正确的去组合这个命令。如：

```
with path: '/usr/local/bin/rbenv/shims:$PATH' do
  execute :ruby, '--version'
end

Will execute:

( PATH=/usr/local/bin/rbenv/shims:$PATH /usr/bin/env ruby --version )
```

相比之下，下面的代码就不会改变命令:

```
with path: '/usr/local/bin/rbenv/shims:$PATH' do
  execute 'ruby --version'
end

# Will execute, without mapping the environmental variables, or querying the command map:

ruby --version
```

# 参考
http://stackoverflow.com/questions/6018591/capistrano-deploy-with-thin-servers
https://github.com/jesson/capistrano3-nginx_unicorn
