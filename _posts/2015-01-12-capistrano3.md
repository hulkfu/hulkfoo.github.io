---
layout: post
title: Capistrano 3
permalink: cap3
---
Capistrano是一个通过ssh在远程Server上部署Web应用的工具。

主要过程是：

* ssh到server，上传git脚本
* 如有需要，创建所需要文件夹，包括：shared和releases
* 从git拉取项目代码到repo目录里并更新
* 在release里创建当前版本的目录，比如20161008094807
* 使用“git archive master | tar -x -f - -C”命令来从repo解压最新代码到上面创建的发布文件夹里
* 把当前repo的最新head存到REVISION里，这个REVISION会保存在current目录里
* 在releases目录里用绝对路径创建指向最新版本的soft link，名为current
* 把上面的current移到应用根目录里

通过以上步骤，就把最新代码发布成功，并保留了之前的代码。

什么重启Nginx、更新数据库的事情，就需要通过HOOK来在Server上执行了。

# 使用

## 安装

Gemfile：

```
group :development do
  gem "capistrano", "~> 3.4"
end
```

然后用bundler安装：

```
$ bundle install
```

## “Capify” 项目

```
$ bundle exec cap install
```

会生成如下文件：

```
├── Capfile
├── config
│   ├── deploy
│   │   ├── production.rb
│   │   └── staging.rb
│   └── deploy.rb
└── lib
    └── capistrano
            └── tasks
```

config/deploy.rb文件是全局的配置文件，然后deploy目录里配置的是对应的各个发布版本的。

## 主要配置变量
配置完这些变量，就能实现基本的发布流程啦！

### :application
应用的名字。

### :deploy_to
default: -> { "/var/www/#{fetch(:application)}" }

应用在远程Server发布的路径。

### :scm
default: :git

代码管理软件，现在支持:git, :hg和:svn。

### :repo_url
代码的URL地址。

比如：

set :repo_url, 'git@example.com:me/my_repo.git' for a git repo located in /home/git/me

如果不是标准ssh端口需要指明其端口：

set :repo_url, 'ssh://git@example.com:30000/~/me/my_repo.git'

### :branch
default: 'master'

要发布的代码分支。

### :repo_path
default: -> { "#{fetch(:deploy_to)}/repo" }

在远程Server上，代码仓库存放的位置，一般不需要改动。

### :repo_tree
default: None. 所有代码都会被发布。

需要发布的代码的子目录。

### :linked_files
default: []

在发布时，只会被链接的文件，它们在各个版本间共享，比如database.yml。

### :linked_dirs
default: []

在发布时，只会被链接的文件夹，它们在各个版本间共享，比如上传文件夹。

### :default_env
default: {}

执行命令时的默认环境shell。

比如，加入rbenv环境：

```ruby
set :default_env, { path: '/opt/rbenv/shims:opt/rbenv/bin:$PATH' }
```

### :keep_releases
default: 5

保留的最后发布版本数。

### :tmp_dir
default: '/tmp'

在发布时用于临时存储数据的文件夹。

如果和别人共享Server，可能需要设置，比如：/home/user/tmp/capistrano。

### :local_user
default: -> { Etc.getlogin }

当前机器的用户名，用来更新发布日志。

### :pty
default: false
Used in SSHKit.

### :log_level
default: :debug
Used in SSHKit.

### :format
default: :pretty
Used in SSHKit.

## 入口

### 变量

变量用set设置，fetch读取：

```ruby
set :application, 'MyLittleApplication'

# use a lambda to delay evaluation
set :application, -> { "SomeThing_#{fetch :other_config}" }
```

可以在任何时候获取设置：

```ruby
fetch :application
# => "MyLittleApplication"

fetch(:special_thing, 'some_default_value')
# will return the value if set, or the second argument as default value
```

### 数组

append方法向array中添加元素，remote则是删除：

```ruby
append :linked_dirs, ".bundle", "tmp"

remove :linked_dirs, ".bundle", "tmp"
```

### 用户输入

可以用ask来需求用户的输入。

```ruby
# used in a configuration
set :database_name, ask('Enter the database name:')

# used in a task
desc "Ask about breakfast"
task :breakfast do
  ask(:breakfast, "pancakes")
  on roles(:all) do |h|
    execute "echo \"$(whoami) wants #{fetch(:breakfast)} for breakfast!\""
  end
end
```

可以在输入密码时通过 echo: false 来设置不显示输入：

```ruby
set :database_password, ask('Enter the database password:', 'default', echo: false)
```

也可以设置默认值：

```ruby
ask(:database_encoding, 'UTF-8')

fetch(:database_encoding)
```

## 用Cap部署后服务器上的目录

```
├── current -> /var/www/my_app_name/releases/20150120114500/
├── releases
│   ├── 20150080072500
│   ├── 20150090083000
│   ├── 20150100093500
│   ├── 20150110104000
│   └── 20150120114500
├── repo
│   └── <VCS related data>
├── revisions.log
└── shared
    └── <linked_files and linked_dirs>
```

current是一个链接，它指向最新的发布版本。当发布完成后，这个链接会自动更新，失败的话就不变。

releases文件夹里是以时间戳命名的版本文件夹。

repo里保存的是版本控制系统等仓库。如果是git的话，就是git的原始仓库，里面有objects, refs等。

revisions.log文件是用来记录每次发布和回滚等。记录每次操作等电脑用户名和时间，还有版本控制系统
等分支等信息。

shared文件夹里面包含着 linked_files 和 linked_dirs，它们会在每次发布后链接到对应的文件或
文件夹。这些数据在各个版本间保存不变且共享，比如数据库配置文件，静态文件或用户上传的文件等。

## 任务
Cap等任务是基于Rake的，只是定义了些类方法。可以在 **lib/capistrano/tasks** 目录下
新建文件定义的*.rake文件，也可以在 **Capfile** 或 **config/deploy.rb** 里定义简单任务。

可以用 cap -T 来查看定义的任务。

### 远程任务

```ruby
server 'example.com', roles: [:web, :app]
server 'example.org', roles: [:db, :workers]
desc "Report Uptimes"
task :uptime do
  on roles(:all) do |host|
    execute :any_command, "with args", :here, "and here"
    info "Host #{host} (#{host.roles.to_a.join(', ')}):\t#{capture(:uptime)}"
  end
end
```

注意： execute(:bundle, :install) and execute('bundle install')是不一样的。

execute()有一个巧妙的方法。比如：

```ruby
within './directory' { execute(:bundle, :install) }
```

excute()的第一个参数是没有空格的字符串，这样传给SSHKit::CommandMap后就会有一系列的强大功能。

当定一个参数里面有空格，不管是Capistrano还是SSHKit都不能准确的进行判断，因此不能在任何上下文中执行或进行命令映射，就是说within(){}, with(), as()等都是无效的。

### 本地任务
通过run_locally块，可以让任务在本地执行：

```ruby
desc 'Notify service of deployment'
task :notify do
  run_locally do
    with rails_env: :development do
      rake 'service:notify'
    end
  end
end
```

或者按照ruby或rake语法：

```ruby
desc 'Notify service of deployment'
task :notify do
  %x('RAILS_ENV=development bundle exec rake "service:notify"')
end

desc "Notify service of deployment"
task :notify do
   sh 'RAILS_ENV=development bundle exec rake "service:notify"'
end
```

## Before/After 钩子

发布的流程的任意阶段，嵌入定义好的任务。

流程见下：

```
deploy:starting    - start a deployment, make sure everything is ready
deploy:started     - started hook (for custom tasks)
deploy:updating    - update server(s) with a new release
deploy:updated     - updated hook
deploy:publishing  - publish the new release
deploy:published   - published hook
deploy:finishing   - finish the deployment, clean up everything
deploy:finished    - finished hook
```


```ruby
# call an existing task
before :starting, :ensure_user

after :finishing, :notify


# or define in block
before :starting, :ensure_user do
  #
end

after :finishing, :notify do
  #
end
```

可以利用rake的机制在使用前创建文件：

```ruby
desc "Create Important File"
file 'important.txt' do |t|
  sh "touch #{t.name}"
end

desc "Upload Important File"
task :upload => 'important.txt' do |t|
  on roles(:all) do
    upload!(t.prerequisites.first, '/tmp')
  end
end
```

通过invoke()来调用其他任务：

```ruby
namespace :example do
  task :one doWhere calling on the same task name, executed in order of inclusion
  task :two do
    invoke "example:one"
    on roles(:all) { info "Two" }
  end
end
```

# ignore on Server

有时我们不想让git目录里的所有文件发布到服务器上，如相关文档，那么就使用 **.gitattributes** 吧。

在根目录下创建.gitattributes文件，里面如下：

```
config/deploy/deploy.rb   export-ignore
config/deploy/            export-ignore
```

不过，cap默认就不会发布deploy相关的文件或文件夹。

# 代码分析

代码很清晰，分为配置和任务。在lib中，除了tasks定义的任务外，dsl和templete就是为configuration服务的。

在模板中，通过dsl来设置服务器的信息。

set设置，fetch获得（map的通过key获得value的方法）。

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

```ruby
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

info()，显示字符串。

### ssh方法

它们定义在backends/netssh.rb里。

test()，加入测试参数，然后execute。

capture()，捕获命令执行的结果为string。

execute()，执行一个shell命令。

upload!()，使用scp上传。

download!()，使用scp下载。

### Command Map —— 命令映射

命令映射，为了解决SSH和Server上的环境变量不同的问题，比如path变量。

可以使用with()方法来设置环境变量，它以一个hash作为参数，如：

```ruby
with path: '/usr/local/bin/rbenv/shims:$PATH' do
  execute :ruby, '--version'
end
```

将会被这样执行：

```
PATH=/usr/local/bin/rbenv/shims:$PATH /usr/bin/env ruby --version
```

而如果将命令后有空格：

```ruby
with path: '/usr/local/bin/rbenv/shims:$PATH' do
  execute 'ruby --version'
end
```

就不会改变环境变量，就是原样执行：

```
ruby --version
```

更常见的是做法是改变command map，它就是一个基于Hash的类型。

默认的前缀是 **/usr/bin/env** 比如：

```ruby
puts SSHKit.config.command_map[:ruby]
# => /usr/bin/env ruby
```

可以直接修改：

```ruby
SSHKit.config.command_map[:rake] = "/usr/local/rbenv/shims/rake"
puts SSHKit.config.command_map[:rake]
# => /usr/local/rbenv/shims/rake
```

也可以添加前缀：

```ruby
SSHKit.config.command_map.prefix[:rake].push("bundle exec")
puts SSHKit.config.command_map[:rake]
# => bundle exec rake

SSHKit.config.command_map.prefix[:rake].unshift("/usr/local/rbenv/bin exec")
puts SSHKit.config.command_map[:rake]
# => /usr/local/rbenv/bin exec bundle exec rake
```

当然也可以覆盖重写command map，虽然不太明智，因为：

```ruby
SSHKit.config.command_map = Hash.new do |hash, command|
  hash[command] = "/usr/local/rbenv/shims/#{command}"
end
```

# 感想
用了Capistrno等自动化部署工具，才会发现部署原来可以如此的方便。

发现重复的事情，然后用代码去流程化，这样就能构建出越来越省力的系统，从而能够去干更重要的事情。

# 参考
http://stackoverflow.com/questions/6018591/capistrano-deploy-with-thin-servers
https://github.com/jesson/capistrano3-nginx_unicorn
