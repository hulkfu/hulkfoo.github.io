---
layout: post
title: Ruby中的线程和进程
---

线程间能够共享变量，所以需要同步等操作。

进程间互相独立，但是可以返回结果给父进程。

# 线程

```ruby
require 'net/http'

pages = %w( www.rubycentral.com
            www.awl.com
            www.pragmaticprogrammer.com
           )

threads = []

for page in pages
  threads << Thread.new(page) { |myPage|

    h = Net::HTTP.new(myPage, 80)
    puts "Fetching: #{myPage}"
    resp, data = h.get('/', nil )
    puts "Got #{myPage}:  #{resp.message}"
  }
end

threads.each { |aThread|  aThread.join }
```

说明：

* Thread.new 后的参数会传到块里，这样块就能单独保有变量而不会与其它线程冲突了。
* join方法，从字面上理解就是“我加入了，等我！”。所以等调用join的线程执行完和时间到指定的等待时间，
这个方法才会返回，使后续代码继续执行。

## 线程变量
线程的定义也是一个块，所以它们共用scope里的变量，块里的是单独变量。

那么该如何访问每个线程的内部变量？thread-local variables.

可以把线程实例当作一个hash，然后读写key-value.

如下，所有线程都有一个mycount变量可以被外面访问：

```ruby
count = 0
arr = []
10.times do |i|
  arr[i] = Thread.new {
    sleep(rand(0)/10.0)
    Thread.current["mycount"] = count
    count += 1
  }
end
arr.each {|t| t.join; print t["mycount"], ", " }
puts "count = #{count}"
```

输出：

```
0, 5, 9, 6, 7, 4, 1, 3, 8, 2, count = 10
```

## 互斥锁

```ruby
#!/usr/bin/ruby
require 'thread'

count1 = count2 = 0
difference = 0
counter = Thread.new do
   loop do
      count1 += 1
      count2 += 1
   end
end
spy = Thread.new do
   loop do
      difference += (count1 - count2).abs
   end
end
sleep 1
puts "count1 :  #{count1}"
puts "count2 :  #{count2}"
puts "difference : #{difference}"
```


```ruby
#!/usr/bin/ruby
require 'thread'
mutex = Mutex.new

count1 = count2 = 0
difference = 0
counter = Thread.new do
   loop do
      mutex.synchronize do
         count1 += 1
         count2 += 1
      end
    end
end
spy = Thread.new do
   loop do
       mutex.synchronize do
          difference += (count1 - count2).abs
       end
   end
end
sleep 1
mutex.lock
puts "count1 :  #{count1}"
puts "count2 :  #{count2}"
puts "difference : #{difference}"
```

其实在2.0以上，执行出来difference都是0，但用锁后效率确实低了些。

用Mutex的实例变量建立互斥区，lock的话就是互斥锁，不用lock而用ConditionVariable的实例变量
来声明条件了就是条件锁，如下：

## 条件锁

```ruby
require 'thread'
mutex = Mutex.new
cv = ConditionVariable.new
a = Thread.new {
  mutex.synchronize {
    puts "A: I have critical section, but will wait for cv"
    cv.wait(mutex)
    puts "A: I have critical section again! I rule!"
  }
}

puts "(Later, back at the ranch...)"

b = Thread.new {
  mutex.synchronize {
    puts "B: Now I am critical, but am done with cv"
    cv.signal
    puts "B: I am still critical, finishing up"
  }
}
a.join
b.join
```

结果：

```
(Later, back at the ranch...)
A: I have critical section, but will wait for cv
B: Now I am critical, but am done with cv
B: I am still critical, finishing up
A: I have critical section again! I rule!
```

ConditionVariable需要在Mutex里使用，因为它其实也是打开锁的一种方式，只是可以指定在什么条件下打开，
更灵活。

# 进程

## Kernel::system

```ruby
system("tar xzf test.tgz")

→ tar: test.tgz: Cannot open:
 No such file or directory
tar: Error is not recoverable: exiting now
tar: Child returned status 2
tar: Error exit delayed from previous errors
false

result = `date`
puts result
```

直接system，参数是命令，那么就会执行，成功返回true，失败返回false，而命令的结果会直接显示出来。

用``包着执行的话，就会把返回结果存进变量里。

## IO.popen

使用IO.popen，就是打开了一条管道pipe，那么就可以与其交互了。


```ruby
cat = IO.popen("cat", "w+")
cat.puts "ice cream after they go to bed"
cat.close_write
puts cat.gets
```

IO.popen回打开一个新的irb：

```ruby
pipe = IO.popen("-","w+")
if pipe
  pipe.puts "Get a job!"
  $stderr.puts "Child says '#{pipe.gets.chomp}'"
else
  $stderr.puts "Dad says '#{gets.chomp}'"
  puts "OK"
end
```

结果：

```
Dad says 'Get a job!'
Child says 'OK'
```


## 独立子进程

有时需要开个子进程做工作，然后主线程继续，就可以使用fork和exec了。

其实system()和IO.popen()方法，也是通过fork和exec实现的。

exec的作用就是让当前进程执行新的程序，替换掉原来的。

```ruby
exec("du ./") unless fork # nil是子进程
puts "main wait."  # 主进程继续
pid = Process.wait
puts "Process #{pid} end."
puts "main end."
```

如果需要监听子进程是否结束，可以trap "CLD"信号：

```ruby
trap("CLD") {
  pid = Process.wait
  puts "Child pid #{pid}: terminated"
  exit
}

exec("sort testfile > output.txt") if fork == nil

# do other stuff...
```

结果：

Child pid 31842: terminated

当然，在上面不管popen还是fork，使用块操作更方便。

IO.popen works with a block in pretty much the same way as File.open does. Pass popen a command, such as date, and the block will be passed an IO object as a parameter.

```ruby
IO.popen ("date") { |f| puts "Date is #{f.gets}" }
```

执行完后，pipe会自动关闭，结果：

```
Date is Sun Jun  9 00:08:50 CDT 2002
```

如果是fork，那么块里的就是子进程，块外是主进程：

```ruby
fork do
  puts "In child, pid = #$$"
  exit 99
end
pid = Process.wait
puts "Child terminated, pid = #{pid}, exit code = #{$? >> 8}"
```

结果：

In child, pid = 31849
Child terminated, pid = 31849, exit code = 99


# 参考
* http://phrogz.net/programmingruby/tut_threads.html
* http://www.tutorialspoint.com/ruby/ruby_multithreading.htm
