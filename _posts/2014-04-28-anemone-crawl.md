---
layout: post
title: anemone爬虫代码分析
---

anemone是用ruby写的一个爬虫，[代码托管在github上](https://github.com/chriskite/anemone)。

# 爬取
爬取的关键算法就在：**从一个开始链接获得更多的可用链接**。

主要爬取功能集中在lib目录下的：

* core.rb
* tentacle.rb
* http.rb
* page.rb

core维护了links和pages两个线程安全队列：links保存待爬取的url，pages保存从url获得的网页数据。

core开始会启动多个tentacle，然后传入第一个起始url，之后tentacle开始工作，通过http获得page，core从page中获得更多links供tentacle爬取，直到所有links都爬取过。

tentacle正如它的英文意思：“触手”，而core是anemone————“海葵”，将它的触手们伸到互联网里去爬取信息。

下面是爬取的主要代码，还有细节代码如判断重定向次数、可选参数处理等没有细说。

## Core
通过使用[Queue](http://ruby-doc.org/stdlib-1.9.3/libdoc/thread/rdoc/Queue.html)类保证进程间同步，从而多线程完成爬取。

link_queue是典型的生成者-消费者模式，从一个页面一级一级爬，将page结果保存到page_queue里，然后待爬取链接保存到link_queue里。

```ruby
#
# 执行爬取
#
def run
  # 处理可选参数
  process_options

  @urls.delete_if { |url| !visit_link?(url) }
  return if @urls.empty?

  # 多线程正是通过这两个queue同步的
  link_queue = Queue.new
  page_queue = Queue.new

  # 创建爬取进程，开始爬取link_queue里的url，并page存到page_queue里。
  @opts[:threads].times do
    @tentacles << Thread.new { Tentacle.new(link_queue, page_queue, @opts).run }
  end

  # 载入爬取的url后，“触手”们就开始工作了。因为Queue在deq时为空时会默认等待。
  @urls.each{ |url| link_queue.enq(url) }

  loop do
    # 处理page信息
    page = page_queue.deq
    @pages.touch_key page.url
    puts "#{page.url} Queue: #{link_queue.size}" if @opts[:verbose]
    do_page_blocks page
    page.discard_doc! if @opts[:discard_page_bodies]

    # 获得这个页面上的所有链接并压入link_queue队列供爬取
    links = links_to_follow page
    links.each do |link|
      link_queue << [link, page.url.dup, page.depth + 1]
    end
    @pages.touch_keys links

    @pages[page.url] = page

    # 如果爬取完，停止线程
    if link_queue.empty? and page_queue.empty?
      until link_queue.num_waiting == @tentacles.size
        Thread.pass
      end
      if page_queue.empty?
        # 以:END结尾
        @tentacles.size.times { link_queue << :END }
        break
      end
    end
  end

  # 等待所有进程结束
  @tentacles.each { |thread| thread.join }
  do_after_crawl_blocks
  self
end

#
# 从所给page返回一个没有被爬取过的链接数组，如果通过设置focus_crawl块，那么将返回块里的处理数据。
#
def links_to_follow(page)
  links = @focus_crawl_block ? @focus_crawl_block.call(page) : page.links
  links.select { |link| visit_link?(link, page) }.map { |link| link.dup }
end

#
# 如果一个link没有被爬取过，且没有被skip_link跳过，没有被robots.txt文件禁止，没有操过最大深度，那么返回true，否则返回false。
#
def visit_link?(link, from_page = nil)
  !@pages.has_page?(link) &&
  !skip_link?(link) &&
  !skip_query_string?(link) &&
  allowed(link) &&
  !too_deep?(from_page)
end

#
# 传入一个处理page里要返回的链接过滤的块，这个块需要返回URI数组。
#
def focus_crawl(&block)
  @focus_crawl_block = block
  self
end
```

## Tentacle
一个执行抓取的可执行单位。

```ruby
#
# 从@link_queue获得links开始爬取，并将爬取内容存入到@page_queue里
#
def run
  loop do
    # 从共享的link queue中取出链接
    link, referer, depth = @link_queue.deq

    break if link == :END

    # 抓取指定link的内容
    @http.fetch_pages(link, referer, depth).each { |page| @page_queue << page }

    delay
  end
end
```

## HTTP
HTTP类主要执行数据抓取的任务。

```ruby
#
# 从访问url返回的请求来创建包括跳转的pages。
#
def fetch_pages(url, referer = nil, depth = nil)
  begin
    url = URI(url) unless url.is_a?(URI)
    pages = []
    # 抓取url，并用结果数据初始化新Page，然后返回
    get(url, referer) do |response, code, location, redirect_to, response_time|
      pages << Page.new(location, :body => response.body.dup,
                                  :code => code,
                                  :headers => response.to_hash,
                                  :referer => referer,
                                  :depth => depth,
                                  :redirect_to => redirect_to,
                                  :response_time => response_time)
    end

    return pages
  rescue Exception => e
    if verbose?
      puts e.inspect
      puts e.backtrace
    end
    return [Page.new(url, :error => e)]
  end
end
```

## Page
Page类主要是对爬取内容的整理，对html的处理使用了Nokogiri。

```ruby
#
# 获得页面上所有本网站的不同的 A tag HREFs 数组
#
def links
  return @links unless @links.nil?
  @links = []
  return @links if !doc

  # 使用xPath语法找到本页的所有链接
  doc.search("//a[@href]").each do |a|
    u = a['href']
    next if u.nil? or u.empty?
    abs = to_absolute(u) rescue next
    # 判断是否是要爬取网站的链接
    @links << abs if in_domain?(abs)
  end
  # 去重
  @links.uniq!
  @links
end
```

# 存储
在core调用crawl方法时，会先调用process_options来处理可选参数，其中传入storage即可指定存储方式，它需要传入一个storage adapter，通过Anemone::Storage里工厂方法来生成。

```ruby
# lib/core.rb

def process_options
  @opts = DEFAULT_OPTS.merge @opts
  @opts[:threads] = 1 if @opts[:delay] > 0
  storage = Anemone::Storage::Base.new(@opts[:storage] || Anemone::Storage.Hash)
  @pages = PageStore.new(storage)
  @robots = Robotex.new(@opts[:user_agent]) if @opts[:obey_robots_txt]

  freeze_options
end
```

然后Anemone::Storage::Base通过传入的adapter来初始化storage。这里可以把Anemone::Storage::Base类理解为一个接口，它的初始化也主要是检查adapter是否含有存储所需要的方法的。

这里的storage主要通过Anemone::PageStore类来存储pages，即爬取的页面，按照url:page存到数据库中，其中page如果不是存在hash中就用Marshal.dump方法序列化。

从而为判断一个url是否爬过、爬虫结束后数据处理等提供支撑。

```ruby
# lib/storage/base.rb

module Anemone
  module Storage
    class Base

      def initialize(adapter)
        @adap = adapter

        # verify adapter conforms to this class's methods
        methods.each do |method|
          if !@adap.respond_to?(method.to_sym)
            raise "Storage adapter must support method #{method}"
          end
        end
      end

      # 接口方法的定义
      ...
    end
  end
end
```

# Bin
作为一个gem，它也为我们提供了方便的命令行工具，可以直接用来使用，同时也是在代码中如何使用anemone的例子。

如果我们执行

```ruby
anemone count http://www.example.com
```

那么首先到bin目录里的anemone可执行文件文件：

```ruby
#!/usr/bin/env ruby
require 'anemone/cli'

Anemone::CLI::run
```

然后执行CLI::run，从输入的AVGV取出第一个参数即count，然后通过load执行count.rb。

```ruby
# lib/cli.rb

module Anemone
  module CLI
    COMMANDS = %w[count cron pagedepth serialize url-list]

    def self.run
      command = ARGV.shift

      if COMMANDS.include? command
        load "anemone/cli/#{command.tr('-', '_')}.rb"
      else
        puts <<-INFO
Anemone is a web spider framework that can collect
useful information about pages it visits.

Usage:
  anemone <command> [arguments]

Commands:
  #{COMMANDS.join(', ')}
INFO
      end
    end
  end
end
```

count.rb就是一个脚本文件了，它require 'anemone'，然后取出剩下的ARGV的第一个参数，就是要爬取的链接地址了。

```ruby
# lib/cli/count.rb

require 'anemone'

begin
  # make sure that the first option is a URL we can crawl
  url = URI(ARGV[0])
rescue
  puts <<-INFO
Usage:
  anemone count <url>

Synopsis:
  Crawls a site starting at the given URL and outputs the total number
  of unique pages on the site.
INFO
  exit(0)
end

Anemone.crawl(url) do |anemone|
  anemone.after_crawl do |pages|
    puts pages.uniq!.size
  end
end
```

因为在执行爬取前会先执行传入的块，且块的参数就是当前的core实例，因此可以直接在块里执行相关方法，这样看上去很优雅，把所有关于它的操作都集中在了它的块了。

```ruby
#
# Convenience method to start a new crawl
#
def self.crawl(urls, opts = {})
  self.new(urls, opts) do |core|
    yield core if block_given?
    core.run
  end
end
```

# 测试
最后但是最重要的，如果一个工程没有好的测试代码，它注定不能被长久维护，不能被社区接受。

Anemone使用[RSpec](http://rubyspec.org/)作为测试框架，[fakeweb](https://github.com/chrisk/fakeweb)来模仿web服务器的请求返回。

spec_helper.rb文件会被每个测试脚本require。

```ruby
require 'rubygems'
require 'bundler/setup'
require 'fakeweb'
require File.dirname(__FILE__) + '/fakeweb_helper'

$:.unshift(File.dirname(__FILE__) + '/../lib/')
require 'anemone'

SPEC_DOMAIN = 'http://www.example.com/'
```

上面的$:是$LOAD_PATH的缩写，那一行代码会把anemone的lib目录放$LOAD_PATH的队首。这样当执行require或load时，它们的优先级就最高。

因为ruby载入的默认顺序是ruby library、/usr/local/lib/ruby、然后才是当前目录，可是要测试的就是当前目录。

下面是core_spec.rb的开头，一个简单的测试用例

```ruby
$:.unshift(File.dirname(__FILE__))
require 'spec_helper'
%w[pstore tokyo_cabinet sqlite3].each { |file| require "anemone/storage/#{file}.rb" }

module Anemone
  describe Core do

    before(:each) do
      FakeWeb.clean_registry
    end

    shared_examples_for "crawl" do
      it "should crawl all the html pages in a domain by following <a> href's" do
        pages = []
        pages << FakePage.new('0', :links => ['1', '2'])
        pages << FakePage.new('1', :links => ['3'])
        pages << FakePage.new('2')
        pages << FakePage.new('3')

        Anemone.crawl(pages[0].url, @opts).should have(4).pages
      end

      # 其他测试
      ...
  end
end
```

很简单，像正常的英语。

# 总结
从分析anemone的代码，能学习到很多东西：

* 多线程
* HTTP协议
* gem的结构
* ruby项目工程的结构
* 鸭子方法的使用
* 数据库的封装
* 项目的测试
* 出错处理

所以说，编程是最容易独自学成高手的，因为金山可以随便出入。关键是要有心人。