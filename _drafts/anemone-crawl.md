---
layout: post
title: anemone爬虫代码分析
---

anemone是用ruby写的一个爬虫，[代码托管在github上](https://github.com/chriskite/anemone)。

# 爬取
爬取的关键算法就在：**从开始页面获得更多的链接**。

主要爬取功能集中在lib目录下的：

* core.rb
* tentacle.rb
* http.rb
* page.rb

代码逻辑比较简单：core调用tentacle，tentacle调用http将数据存到page。

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

# Bin

# 总结
爬虫是我们收集信息的第一步，是件很有意思的事情。