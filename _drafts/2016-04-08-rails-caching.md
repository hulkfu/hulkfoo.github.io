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

## 1.ActiveSupport::Cache::Store
This class provides the foundation for interacting with the cache in Rails. This is an abstract class and you cannot use it on its own. Rather you must use a concrete implementation of the class tied to a storage engine. Rails ships with several implementations documented below.

The main methods to call are read, write, delete, exist?, and fetch. The fetch method takes a block and will either return an existing value from the cache, or evaluate the block and write the result to the cache if no value exists.



## 2.ActiveSupport::Cache::MemoryStore
This cache store keeps entries in memory in the same Ruby process. The cache store has a bounded size specified by the :size options to the initializer (default is 32Mb). When the cache exceeds the allotted size, a cleanup will occur and the least recently used entries will be removed.

config.cache_store = :memory_store, { size: 64.megabytes }
If you're running multiple Ruby on Rails server processes (which is the case if you're using mongrel_cluster or Phusion Passenger), then your Rails server process instances won't be able to share cache data with each other. This cache store is not appropriate for large application deployments, but can work well for small, low traffic sites with only a couple of server processes or for development and test environments.




## ActiveSupport::Cache::FileStore
This cache store uses the file system to store entries. The path to the directory where the store files will be stored must be specified when initializing the cache.

config.cache_store = :file_store, "/path/to/cache/directory"
With this cache store, multiple server processes on the same host can share a cache. Servers processes running on different hosts could share a cache by using a shared file system, but that set up would not be ideal and is not recommended. The cache store is appropriate for low to medium traffic sites that are served off one or two hosts.

Note that the cache will grow until the disk is full unless you periodically clear out old entries.

This is the default cache store implementation.


## ActiveSupport::Cache::MemCacheStore
This cache store uses Danga's memcached server to provide a centralized cache for your application. Rails uses the bundled dalli gem by default. This is currently the most popular cache store for production websites. It can be used to provide a single, shared cache cluster with very high performance and redundancy.

When initializing the cache, you need to specify the addresses for all memcached servers in your cluster. If none is specified, it will assume memcached is running on the local host on the default port, but this is not an ideal set up for larger sites.

The write and fetch methods on this cache accept two additional options that take advantage of features specific to memcached. You can specify :raw to send a value directly to the server with no serialization. The value must be a string or number. You can use memcached direct operation like increment and decrement only on raw values. You can also specify :unless_exist if you don't want memcached to overwrite an existing entry.




## ActiveSupport::Cache::EhcacheStore
If you are using JRuby you can use Terracotta's Ehcache as the cache store for your application. Ehcache is an open source Java cache that also offers an enterprise version with increased scalability, management, and commercial support. You must first install the jruby-ehcache-rails3 gem (version 1.1.0 or later) to use this cache store.




## ActiveSupport::Cache::NullStore
This cache store implementation is meant to be used only in development or test environments and it never stores anything. This can be very useful in development when you have code that interacts directly with Rails.cache, but caching may interfere with being able to see the results of code changes. With this cache store, all fetch and read operations will result in a miss.

config.cache_store = :null_store
2.8 Custom Cache Stores
You can create your own custom cache store by simply extending ActiveSupport::Cache::Store and implementing the appropriate methods. In this way, you can swap in any number of caching technologies into your Rails application.

To use a custom cache store, simple set the cache store to a new instance of the class.

config.cache_store = MyCacheStore.new



## Cache Keys
The keys used in a cache can be any object that responds to either :cache_key or to :to_param. You can implement the :cache_key method on your classes if you need to generate custom keys. Active Record will generate keys based on the class name and record id.

You can use Hashes and Arrays of values as cache keys.

```ruby
# This is a legal cache key
Rails.cache.read(site: "mysite", owners: [owner_1, owner_2])
```

The keys you use on Rails.cache will not be the same as those actually used with the storage engine. They may be modified with a namespace or altered to fit technology backend constraints. This means, for instance, that you can't save values with Rails.cache and then try to pull them out with the memcache-client gem. However, you also don't need to worry about exceeding the memcached size limit or violating syntax rules.




# Conditional GET support
Conditional GETs are a feature of the HTTP specification that provide a way for web servers to tell browsers that the response to a GET request hasn't changed since the last request and can be safely pulled from the browser cache.

They work by using the HTTP_IF_NONE_MATCH and HTTP_IF_MODIFIED_SINCE headers to pass back and forth both a unique content identifier and the timestamp of when the content was last changed. If the browser makes a request where the content identifier (etag) or last modified since timestamp matches the server's version then the server only needs to send back an empty response with a not modified status.

It is the server's (i.e. our) responsibility to look for a last modified timestamp and the if-none-match header and determine whether or not to send back the full response. With conditional-get support in Rails this is a pretty easy task:

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
Instead of an options hash, you can also simply pass in a model, Rails will use the updated_at and cache_key methods for setting last_modified and etag:

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
If you don't have any special response processing and are using the default rendering mechanism (i.e. you're not using respond_to or calling render yourself) then you've got an easy helper in fresh_when:

class ProductsController < ApplicationController

  # This will automatically send back a :not_modified if the request is fresh,
  # and will render the default template (product.*) if it's stale.

  def show
    @product = Product.find(params[:id])
    fresh_when last_modified: @product.published_at.utc, etag: @product
  end
end





# 常用Caching 方案 ———— Memocached + Dalli

## [memcached](https://memcached.org/)

## [Dalli](https://github.com/petergoldstein/dalli)

Dalli is a high performance pure Ruby client for accessing memcached servers. It works with memcached 1.4+ only as it uses the newer binary protocol. It should be considered a replacement for the memcache-client gem.

# 参考

* http://guides.rubyonrails.org/caching_with_rails.html
* https://signalvnoise.com/posts/3113-how-key-based-cache-expiration-works
