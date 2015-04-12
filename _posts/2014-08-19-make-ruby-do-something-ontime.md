---
layout: post
title: 让ruby定时按点办事
---

定时任务是个好东西，让电脑定时按点为您办事。

# [whenever](https://github.com/javan/whenever)

它是对Linux cron语法的封装，把ruby代码翻译成cron，通过cron执行的，所以只能在Linux下使用。

[RailsCasts参考](http://railscasts.com/episodes/164-cron-in-ruby/)。

通过下面命令会在当前文件夹的config目录下建立一个schedule.rb文件。

```
cd /apps/my-great-project
wheneverize .
```

编写代码：

```ruby
every 3.hours do
  runner "MyModel.some_process"
  rake "my:rake:task"
  command "/usr/bin/my_great_command"
end

every 1.day, :at => '4:30 am' do
  runner "MyModel.task_to_run_at_four_thirty_in_the_morning"
end

every :hour do # Many shortcuts available: :hour, :day, :month, :year, :reboot
  runner "SomeModel.ladeeda"
end

every :sunday, :at => '12pm' do # Use any day of the week or :weekend, :weekday
  runner "Task.do_something_great"
end

every '0 0 27-31 * *' do
  command "echo 'you can use raw cron syntax too'"
end

# run this task only on servers with the :app role in Capistrano
# see Capistrano roles section below
every :day, :at => '12:20am', :roles => [:app] do
  rake "app_server:task"
end
```

默认定义了4种任务：

```
job_type :command, ":task :output"
job_type :rake,    "cd :path && :environment_variable=:environment bundle exec rake :task --silent :output"
job_type :runner,  "cd :path && script/rails runner -e :environment ':task' :output"
job_type :script,  "cd :path && :environment_variable=:environment bundle exec script/:task :output"
```

当然也可用job_type定义自己的任务：

```ruby
job_type :awesome, '/usr/local/bin/awesome :task :fun_level'

every 2.hours do
  awesome "party", :fun_level => "extreme"
end
```

其中第一个参数会是:task，之后的就用map来赋值了。

之后用whenever可以生成cron代码，然后用

```
whenever -w
```
就会写到crontab文件了。

因为是基于cron，所以要保证其运行，在ubuntu下：

```
service cron start
```

就启动了cron。

可见，whenever适合做固定的工作，如它默认定义的rake、runner等需要固定做的工作，而不适合在编程时灵活调用。

# [clockwork](https://github.com/tomykaira/clockwork)
clockwork是cron的代替，它不像whenever需要Linux cron的支持，而是通过启动一个长运行的ruby进程来进行调度的，因此它是纯靠Ruby实现的跨平台的。

官方的例子：

```ruby
require 'clockwork'
module Clockwork
  handler do |job|
    puts "Running #{job}"
  end

  # handler receives the time when job is prepared to run in the 2nd argument
  # handler do |job, time|
  #   puts "Running #{job}, at #{time}"
  # end

  every(10.seconds, 'frequent.job')
  every(3.minutes, 'less.frequent.job')
  every(1.hour, 'hourly.job')

  every(1.day, 'midnight.job', :at => '00:00')
end
```

运行：

```
clockwork clock.rb
```

如果在rails里使用，则在require 'clockwork'下加入对环境的引用：

```
require './config/boot'
require './config/environment'
```

clockwork只是负责只是在一个进程中负责定时定点的任务的分发，而不管任务的执行，如有有一个延时任务就会阻碍掉它的进行，所以就需要一个后台任务执行队列来配合，如： Delayed Job, Beanstalk/Stalker, RabbitMQ/Minion, Resque 或者 Sidekiq等。

这样既不会有死锁，还能灵活满足各种需求。

如使用[Stalker](https://github.com/adamwiggins/stalker)，[RailsCasts参考](http://railscasts.com/episodes/243-beanstalkd-and-stalker)：

```ruby
require 'stalker'

module Clockwork
  handler { |job| Stalker.enqueue(job) }

  every(1.hour, 'feeds.refresh')
  every(1.day, 'reminders.send', :at => '01:30')
end
```

也可以使用:thread参数，来在新线程中执行任务：

```
Clockwork.every(1.day, 'run.me.in.new.thread', :thread => true)
```

还可以动态的从数据库中读入，然后执行：

```ruby
require 'clockwork'
require 'clockwork/manager_with_database_tasks'
require_relative './config/boot'
require_relative './config/environment'

module Clockwork

  # required to enable database syncing support
  Clockwork.manager = ManagerWithDatabaseTasks.new

  sync_database_tasks model: MyScheduledTask, every: 1.minute do |instance_job_name|
    # Where your model will acts as a worker:
    id = instance_job_name.split(':').last
    task = MyScheduledTask.find(id)
    task.perform_async

    # Or, e.g. if your queue system just needs job names
    # Stalker.enqueue(instance_job_name)
  end

  [...other tasks if you have...]

end
```

## 注意
* 一个文件里只有一个handler能够运行。
* 在every后面使用block来执行任务的话，就不会再次触发handler了（明确的说只有第一次触发，以后就不触发了）。
* 在做产品时，需要用[god](http://godrb.com/)等进程监视器保证其运行。

# [sidekiq](https://github.com/mperham/sidekiq)

sidekiq是rails官方支持，使用很方便。

它需要[redis](http://www.redis.io/)的支持。

[RailsCasts参考](http://railscasts.com/episodes/366-sidekiq)。

## 在Rails中使用

* 首先在Gemfile里引入：

```
gem 'sidekiq'
```

* 然后在app/works目录下编写work：

```ruby
# app/workers/hard_worker.rb
class HardWorker
  include Sidekiq::Worker

  def perform(name, count)
    puts 'Doing hard work'
  end
end
```

就可以在model中调用了：

```ruby
HardWorker.perform_async('bob', 5)
```

* 最后在Rails app的根目录下让sidekiq开始运行：

```ruby
bundle exec sidekiq
```

## 在Ruby里单独使用
还在研究中。

# [DJ(delayed_job)](https://github.com/collectiveidea/delayed_job)

http://railscasts.com/episodes/171-delayed-job

如其名，就是在后台执行，用着很方便。

## 安装

mongoid下使用：

gem 'delayed_job_mongoid'

active record下用：

gem 'delayed_job_active_record'


## 启动worker
需要启动worker才能运行delay的任务。

### 使用rake

```
rake jobs:work
```

### 在后台执行

1. 需要 [daemons](https://github.com/ghazel/daemons) gem.
2. 使用**rails generate delayed_job**生成脚本文件，Rails 3在script目录下，rails 4在bin目录下。
3. 使用。

```
RAILS_ENV=production script/delayed_job start
RAILS_ENV=production script/delayed_job stop

# 运行2个worker
RAILS_ENV=production script/delayed_job -n 2 start
RAILS_ENV=production script/delayed_job stop

# 使用--queue 或 --queues执行特定的队列
RAILS_ENV=production script/delayed_job --queue=tracking start
RAILS_ENV=production script/delayed_job --queues=mailers,tasks start

# 使用 --pool 来指定一个worker pool。可以对不同的队列多次使用来指定不同的worker。
# 下面的命令会给tracking队列指定一个worker，mailer、tasks两个，其它的两个。
RAILS_ENV=production script/delayed_job --pool=tracking --pool=mailers,tasks:2 --pool=*:2 start

# 执行所有可执行任务，然后退出
RAILS_ENV=production script/delayed_job start --exit-on-complete
# 在前台执行
RAILS_ENV=production script/delayed_job run --exit-on-complete
```

## 在代码中使用
在任何对象上调用**.delay.method(params)**就会使这个method在后台执行。
```
# without delayed_job
@user.activate!(@device)

# with delayed_job
@user.delay.activate!(@device)
```

如果一个方法总是需要在后台执行，可以在声明它后使用#handle_asynchronously来指明。然后可以使用methoud_without_delay来调用原来的方法。

```
class Device
  def deliver
    # long running method
  end
  handle_asynchronously :deliver
end

device = Device.new
device.deliver
```

handle_asynchronously接受run_at，priority参数，并且值可以用Proc定义。

```
class LongTasks
  def send_mailer
    # Some other code
  end
  handle_asynchronously :send_mailer, :priority => 20

  def in_the_future
    # Some other code
  end
  # 5.minutes.from_now will be evaluated when in_the_future is called
  handle_asynchronously :in_the_future, :run_at => Proc.new { 5.minutes.from_now }

  def self.when_to_run
    2.hours.from_now
  end

  class << self
    def call_a_class_method
      # Some other code
    end
    handle_asynchronously :call_a_class_method, :run_at => Proc.new { when_to_run }
  end

  attr_reader :how_important

  def call_an_instance_method
    # Some other code
  end
  handle_asynchronously :call_an_instance_method, :priority => Proc.new {|i| i.how_important }
end
```

## Capistrano 部署后自动运行

```
require "[delayed/recipes](https://github.com/collectiveidea/delayed_job/blob/master/lib/delayed/recipes.rb)"

after "deploy:stop",    "delayed_job:stop"
after "deploy:start",   "delayed_job:start"
after "deploy:restart", "delayed_job:restart"
```

# active job
Rails 4.2的新特性，为以上提供了接口，这样可以同一使用了。

在config/application.rb文件配置要使用的引擎：

```
config.active_job.queue_adapter = :delayed_job
```

# 总结
对比以上，whenever运维用，clockwork单独用，sidekiq 在 Rails里用。

最后Rails开不下去了，出了个active job。

# 参考
* http://www.cnblogs.com/wangyuyu/p/3818826.html