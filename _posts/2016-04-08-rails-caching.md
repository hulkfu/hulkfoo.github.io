---
layout: post
title: Rails 缓存
---

> There are only two hard things in Computer Science:
> cache invalidation and naming things.
>
> — Phil Karlton

缓存的作用就是不变的数据只生成一次，下次直接访问生生成好的数据。

那么问题来了：

1. 怎么判断缓存里信息过期没有？
2. 缓存数据的粒度多大，如何使用？
3. 缓存的存储方法有哪些？

### 问题一，基于key的缓存过期机制

Caching服务就是一个key-value服务：

* 根据key，找到value了，说明cache能用，直接读取。
* 根据key，没有value，说明还没有cache，去Rails栈重新生成数据。

可见这里关键是改变key，而不是更新value，每一个key的value都是死的。那么问题就化解成如何生成key了：

根据要缓存数据的特征及时更新key值，比如总体数目、更新时间等信息，只要能反应出数据变了。

这种基于key的缓存机制，是被动缓存的思想。想想如果基于value，那么维护value是多么的麻烦！

### 问题二，Rails中的种缓存粒度

主要有：

1. View缓存
2. Low-Level缓存，可以理解为model层的
3. SQL缓存
4. HTTP GET缓存


# 基本Caching

## Fragment Caching

* Page Caching：整个页面，不经过Rails栈。
* Action Caching：Action后的内容，通过Rails判断是否符合条件，比如认证。
* Fragment Caching：页面内嵌入的缓存。

从Rails 4开始，Page和Action缓存已经分别移到
actionpack-page_caching和actionpack-action_caching两个gem里了。

所以下面主要说明Fragment Caching。

动态网页都是由很多块动态数据生成的，每一块有共同生命周期的数据可以形成一块Fragment。

除非这块Fragment过期，否则就直接从缓存读取请求的数据，不需要经过Rails栈的生成。

比如缓存所有的Product：

```ruby
<% Order.find_recent.each do |o| %>
  <%= o.buyer.name %> bought <%= o.product.name %>
<% end %>

<% cache do %>
  All available products:
  <% Product.all.each do |p| %>
    <%= link_to p.name, product_url(p) %>
  <% end %>
<% end %>
```

这个cache块将会和action绑定，而且这里面的所以cache都将是同一个名字，所以需要用
action_suffix来区分：


```ruby
<% cache(action: 'recent', action_suffix: 'all_products') do %>
  All available products:
```


可以通过expire_fragment方法作废掉上面的缓存：

```
expire_fragment(controller: 'products', action: 'recent', action_suffix: 'all_products')
```


如果不想将cache块和action绑定，也可以为那个fragment起一个全局的key：

```ruby
<% cache('all_available_products') do %>
  All available products:
<% end %>
```

那么这个fragment将会在ProductsController的所有action里可用，同样可以用expire_fragment过期它：


```ruby
expire_fragment('all_available_products')
```

如果不想手动去过期fragment，可以写一个helper，在这个action更新时过期它：

```ruby
module ProductsHelper
  def cache_key_for_products
    count          = Product.count
    max_updated_at = Product.maximum(:updated_at).try(:utc).try(:to_s, :number)
    "products/all-#{count}-#{max_updated_at}"
  end
end
```

这个方法根据所有products的个数生成一个cache key，然后可以这样使用：


```ruby
<% cache(cache_key_for_products) do %>
  All available products:
<% end %>
```

如果想根据条件来判断是否需要cache，可以使用cache_if：

```ruby
<% cache_if (condition, cache_key_for_products) do %>
  All available products:
<% end %>
```

也可以用一个Active model来作为cache key：


```ruby
<% Product.all.each do |p| %>
  <% cache(p) do %>
    <%= link_to p.name, product_url(p) %>
  <% end %>
<% end %>
```

在这种情况下，一个叫 cache_key 的方法将会被这个实例调用，并返回一个像 products/23-20130109142513
的字符串。这个 cache_key 包含实例的name、id及最后更新的时间戳。因此当这个product更新时，会自动
生成一个新的fragment。

也可以将以上两种策略结合起来，叫 "Russian Doll Caching":

```ruby
<% cache(cache_key_for_products) do %>
  All available products:
  <% Product.all.each do |p| %>
    <% cache(p) do %>
      <%= link_to p.name, product_url(p) %>
    <% end %>
  <% end %>
<% end %>
```

之所以叫"Russian Doll Caching"，是因为它嵌入了多层fragment。优点是，如果只有一个fragment更新了，
其它的可以继续使用缓存。


## Low-Level Caching

有时需要缓存一块数据或查询结果等更低一级的信息，而不是view fragment。

最有效的方法就是使用Rails.cache.fetch方法了，这个方法可以读或写cache。

当只传一个key参数，那么如果fetched就会返回。

如果除了key参数还传了一个代码块，那么如果找不到key对应的value时，就会执行代码块，并将结果出入cache并返回。

举个例子，一个Product实例需要返回competing_price，可以如下实现：

```ruby
class Product < ActiveRecord::Base
  def competing_price
    Rails.cache.fetch("#{cache_key}/competing_price", expires_in: 12.hours) do
      Competitor::API.find_price(id)
    end
  end
end
```

注意：上面使用了 cache_key 方法，它会返回像 products/233-20140225082222765838000/competing_price
这样的字符串，之前说过，它根据model、id、updated_at生成。

当然你也可以实现自己的cache_key方法。


## SQL Caching

Rails能自动缓存SQL查询数据，并自动返回重复的查询结果，比如：


```ruby
class ProductsController < ApplicationController

  def index
    # Run a find query
    @products = Product.all

    ...

    # Run the same query again
    @products = Product.all
  end

end
```

# Cache Stores
Rails对Action和Fragment提供不同的Cache储存方案，Page存储在硬盘里。

可以通过 config.cache_store= 来配置，针对所以的在config/application.rb文件里，针对不同环境的
在config/environments/里各个文件里。

第一个参数是使用的cache store，后面的将会作为参数传给cache store的构造函数。

## ActiveSupport::Cache::Store

这个类提供Rails与cache交互的方法，但它是一个抽象类，需要相应的cache store的具体实现类才能使用。

主要有read, wreite, delete, exist? 和 fetch 方法。Fetch方法能够接受一个block，这样有值的话
就会返回，没有的话就会执行block，然后将结果写入到cache并返回。

下面分别介绍支持的cache store：

### 1. ActiveSupport::Cache::MemoryStore
MemoryStore类型的存储方式会把所有内容存到Ruby的同一个进程里。它有一个:size参数来定义最大缓存内存大小，
默认是32Mb。当cache超过这个值，那么将会执行清理程序，最少使用的内容将被清理。

```ruby
config.cache_store = :memory_store, { size: 64.megabytes }
```

如果跑了多个Rails进程（就像使用mongrel_cluster or Phusion Passenger的情况下），那么这些Rails
进程间将不会共享缓存。这种方式不适合大型应用的，但对浏览小的小网站的开发和测试还是适合的。

### 2. ActiveSupport::Cache::FileStore
FileStore将内容存到文件里，如果需要指定文件路径，需要事先指明：

```ruby
config.cache_store = :file_store, "/path/to/cache/directory"
```

这种方式下，多个Server进程间可以共享同一个缓存。即使不同主机的Servers进程也可以通过文件共享系统
共享同一个缓存，但这是不推荐的。这种方式时候中低流量的部署在一两个主机下的网站。

注意：缓存的大小会一直增加到整个磁盘的大小，除非你清理一下。

这种方式也是默认的缓存方式。


### 3. ActiveSupport::Cache::MemCacheStore
MemCachStore这种放方式使用memcached server来提供集中的缓存服务。

Rails默认使用dalli gem来对接。这也是当下产品部署最流行的方式。

它能够提供一种单一的共享的缓存集群，并且具有很好的性能和冗余。

当初始化这个cache，需要指定memcached servers的地址和端口。如果没有设置，就默认localhost的默认
端口，显然对于大型网络是不理想的。

这种方式下，write和fetch方法接受两个额外的参数。你可以使用:raw参数直接传送给server没有序列化的
数据，但这个值必须是string或数字。可以通过memcached直接操作raw数据，比如加或减。也可以使用
:unless_exist参数使memcached不覆盖已有的内容。


### 4. ActiveSupport::Cache::EhcacheStore
EhcacheStore是在使用JRuyb下使用的，它以Terracotta's Ehcache为后台。


### 5. ActiveSupport::Cache::NullStore
NullStore用在测试和开发环节，如其名，不存任何东西，所有对cache的访问都将miss。

```ruby
config.cache_store = :null_store
```

### 6. 自定义自己的Cache Store
通过扩展ActiveSupport::Cache::Store类，并实现相应的方法，就可以自定义cache store了。

在配置文件那里新建cache store类的实例即可使用了：

```ruby
config.cache_store = MyCacheStore.new
```

## Cache Keys
cache中使用的key可以在任何响应 :cache_key 或 :to_param的对象中生成，你也可以在自己的类定义中定义
:cache_key方法。ActiveRecord是通过类名和id及更新时间来生成cache key的。

You can use Hashes and Arrays of values as cache keys.
可以使用Hash和Array的值作为cache key：

```ruby
# This is a legal cache key
Rails.cache.read(site: "mysite", owners: [owner_1, owner_2])
```

在Rails.cache中的key值将因为不同的后台引擎而不同。这些key值可能会加一个namespace或其它改变。所以
不同的后台，生成的key将不同。


# 有条件的 GET 支持
有条件GETs是HTTP协议的一个功能，它定义一种方法让Server告诉浏览器这次访问的请求与上次的请求相同，所以
可以直接使用浏览器cache的内容，从而减少了HTTP传输。

通过使用HTTP_IF_NONE_MATCH 和 HTTP_IF_MODIFIED_SINCE头及内容的id和时间戳来工作。如果浏览器请求的内容id（即etag）或
上次更新时间戳与服务器上的版本相同，那么服务器只需要返回一个带有没有改变的空相应。

Server通过查看请求的时间戳和 if-none-match 头来决定是否相应全部内容，
在Rails中，conditional-get如下使用：

```ruby
class ProductsController < ApplicationController

  def show
    @product = Product.find(params[:id])

    # If the request is stale according to the given timestamp and etag value
    # (i.e. it needs to be processed again) then execute this block
    if stale?(last_modified: @product.updated_at.utc, etag: @product.cache_key)
      respond_to do |wants|
        # ... normal response processing
      end
    end

    # If the request is fresh (i.e. it's not modified) then you don't need to do
    # anything. The default render checks for this using the parameters
    # used in the previous call to stale? and will automatically send a
    # :not_modified. So that's it, you're done.
  end
end
```

不用hash参数的话，也可以使用实例来代替，Rails会根据updated_at和cache_key来生成
last_modified和etag的：

```ruby
class ProductsController < ApplicationController
  def show
    @product = Product.find(params[:id])

    if stale?(@product)
      respond_to do |wants|
        # ... normal response processing
      end
    end
  end
end
```

如果你不想对相应做特殊处理，使用默认的模板生成，那么可以简单使用 fresh_when：

```ruby
class ProductsController < ApplicationController

  # This will automatically send back a :not_modified if the request is fresh,
  # and will render the default template (product.*) if it's stale.

  def show
    @product = Product.find(params[:id])
    fresh_when last_modified: @product.published_at.utc, etag: @product
  end
end
```

# 常用Caching 方案 —— Memocached + Dalli

## [memcached](https://memcached.org/)

Mac下用brew安装：

```
brew install memcached --with-sasl
```

里面的参数是为了配合[dalli gem](https://github.com/petergoldstein/dalli).

## [Dalli](https://github.com/petergoldstein/dalli)
memcached的Ruby接口。

在Gemfile里添加dalli，然后去环境里配置：

```
config.cache_store = :dalli_store
```

# 参考

* http://guides.rubyonrails.org/caching_with_rails.html
* https://signalvnoise.com/posts/3113-how-key-based-cache-expiration-works
