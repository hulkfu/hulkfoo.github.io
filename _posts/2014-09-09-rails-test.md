---
layout: post
title: Rails Test
---

把经常在做的测试过程抽象成测试用例代码，比如功能函数、操作过程、性能等任何能想到的。

首先亮明观点，我觉得 Rails 自带的 Minitest 和 Fixture 就很好用。

# Minitest
Minitest 是默认的，也挺好用的，而且还简单。

一个例子：

```ruby
require 'test_helper'

# TestCase 继承自 Minitest::Test
class ProjectTest < ActiveSupport::TestCase
  # 定义测试，等同于 def test_the_truth
  test "the truth" do
    assert true, "some message"
  end
end
```

常用测试判断 [assertion](http://docs.seattlerb.org/minitest/Minitest/Assertions.html):

```ruby
assert( test, [msg] )	Ensures that test is true.
assert_not( test, [msg] )	Ensures that test is false.
assert_equal( expected, actual, [msg] )	Ensures that expected == actual is true.
assert_not_equal( expected, actual, [msg] )	Ensures that expected != actual is true.
assert_same( expected, actual, [msg] )	Ensures that expected.equal?(actual) is true.
assert_not_same( expected, actual, [msg] )	Ensures that expected.equal?(actual) is false.
assert_nil( obj, [msg] )	Ensures that obj.nil? is true.
assert_not_nil( obj, [msg] )	Ensures that obj.nil? is false.
assert_empty( obj, [msg] )	Ensures that obj is empty?.
assert_not_empty( obj, [msg] )	Ensures that obj is not empty?.
assert_match( regexp, string, [msg] )	Ensures that a string matches the regular expression.
assert_no_match( regexp, string, [msg] )	Ensures that a string doesn't match the regular expression.
assert_includes( collection, obj, [msg] )	Ensures that obj is in collection.
assert_not_includes( collection, obj, [msg] )	Ensures that obj is not in collection.
assert_in_delta( expected, actual, [delta], [msg] )	Ensures that the numbers expected and actual are within delta of each other.
assert_not_in_delta( expected, actual, [delta], [msg] )	Ensures that the numbers expected and actual are not within delta of each other.
assert_throws( symbol, [msg] ) { block }	Ensures that the given block throws the symbol.
assert_raises( exception1, exception2, ... ) { block }	Ensures that the given block raises one of the given exceptions.
assert_instance_of( class, obj, [msg] )	Ensures that obj is an instance of class.
assert_not_instance_of( class, obj, [msg] )	Ensures that obj is not an instance of class.
assert_kind_of( class, obj, [msg] )	Ensures that obj is an instance of class or is descending from it.
assert_not_kind_of( class, obj, [msg] )	Ensures that obj is not an instance of class and is not descending from it.
assert_respond_to( obj, symbol, [msg] )	Ensures that obj responds to symbol.
assert_not_respond_to( obj, symbol, [msg] )	Ensures that obj does not respond to symbol.
assert_operator( obj1, operator, [obj2], [msg] )	Ensures that obj1.operator(obj2) is true.
assert_not_operator( obj1, operator, [obj2], [msg] )	Ensures that obj1.operator(obj2) is false.
assert_predicate ( obj, predicate, [msg] )	Ensures that obj.predicate is true, e.g. assert_predicate str, :empty?
assert_not_predicate ( obj, predicate, [msg] )	Ensures that obj.predicate is false, e.g. assert_not_predicate str, :empty?
assert_send( array, [msg] )	Ensures that executing the method listed in array[1] on the object in array[0] with the parameters of array[2 and up] is true, e.g. assert_send [@user, :full_name, 'Sam Smith']. This one is weird eh?
flunk( [msg] )	Ensures failure. This is useful to explicitly mark a test that isn't finished yet.
```

# Fixture
Rails 自带的 Fixture 也是简单好用，不要为了生成几个数据而且引入一个生成框架，而且它还很准确。

比如  <your-rails-app>/test/fixtures/web_sites.yml 文件：

```ruby
rubyonrails:
  id: 1
  name: Ruby on Rails
  url: http://www.rubyonrails.org

google:
  id: 2
  name: Google
  url: http://www.google.com
```

然后在 test_helper.rb 里，Fixtures 会在执行测试前装入数据库：

```ruby
class ActiveSupport::TestCase
  # Setup all fixtures in test/fixtures/*.yml for all tests in alphabetical order.
  fixtures :all
end

```

可以用 Fixture 的文件名和里面实例的名字来引用：

```ruby
web_sites(:rubyonrails)
```

也通过声明 self.use_instantiated_fixtures = true 来实例化 fixtures：

```ruby
# 可以直接用 yml 里的名字来找到数据
test "find" do
  assert_equal "Ruby on Rails", web_sites(:rubyonrails).name
end

# 或者直接用 @实例名
test "find_alt_method_2" do
  assert_equal "Ruby on Rails", @rubyonrails.name
end
```

## 嵌入 erb 动态数据

```ruby
<% 1.upto(1000) do |i| %>
fix_<%= i %>:
  id: <%= i %>
  name: guy_<%= i %>
<% end %>
```

可以在 test_helper.rb 里定义公用的 help：

```ruby
module FixtureFileHelpers
  def file_sha(path)
    Digest::SHA2.hexdigest(File.read(Rails.root.join('test/fixtures', path)))
  end
end
ActiveRecord::FixtureSet.context_class.include FixtureFileHelpers
```

# 数据库清理



# RSpec
在Rails下需要使用[rspec-rails](https://github.com/rspec/rspec-rails)，其官网参考文档很详细。

## 注意
* 把.rspec里的--warning去掉，否则会有一大堆的警告。

# [FactoryGirl](https://github.com/thoughtbot/factory_girl)
如其名，生成model实例的。

如果使用rails，需要用factory_girl_rails这个gem。[这里有入门](https://github.com/thoughtbot/factory_girl/blob/master/GETTING_STARTED.md)。

使用前可以在rspec里配置includeFactoryGirl的DSL，这样就不需要在测试时前面每次都加FactoryGirl了。

```ruby
# RSpec
# spec/support/factory_girl.rb
# 可以在rails_helper.rb里require
RSpec.configure do |config|
  config.include FactoryGirl::Syntax::Methods
end
```

首先定义factory，放在下列目录下时会自动加载：

* test/factories.rb
* spec/factories.rb
* test/factories/*.rb
* spec/factories/*.rb

```ruby
FactoryGirl.define do
  # This will guess the User class
  factory :user do
    first_name "John"
    last_name  "Doe"
    admin false
  end

  # This will use the User class (Admin would have been guessed)
  factory :admin, class: User do
    first_name "Admin"
    last_name  "User"
    admin      true
  end
end
```

然后就可以使用了：

```ruby
# Returns a User instance that's not saved
user = build(:user)

# Returns a saved User instance
user = create(:user)

# Returns a hash of attributes that can be used to build a User instance
attrs = attributes_for(:user)

# Returns an object with all defined attributes stubbed out
stub = build_stubbed(:user)

# Passing a block to any of the methods above will yield the return object
create(:user) do |user|
  user.posts.create(attributes_for(:post))
end

# No matter which strategy is used, it's possible to override the defined attributes by passing a hash:

# Build a User instance and override the first_name property
user = build(:user, first_name: "Joe")
user.first_name
# => "Joe"

```

FactoryGirl还有继承、关联、序列等用法，可以参考上面提到的文档。

## 其它
当使用carriewave时，直接在factories文件里用**File.open**打开文件。如：

```ruby
FactoryGirl.define do
  factory :task do
    name "task"
    attachment File.open(File.join(Rails.root, 'spec', 'support', 'attachments', 'file.txt'))
  end
```

# [capybara](https://github.com/jnicklas/capybara)
它模仿浏览器的行为，所以使用它就像自己在操作浏览器一样。

也因此只有在feature测试中才能使用。

有时capybara提示ElementNotFound，可能是真的没有找到，也可能是其它地方出了问题，然后它只catch到这个错误。此时去浏览器里重现一下场景看看。

# [Jasmine]
纯测试 JavaScript 的功能，即不涉及后端的 Ruby 代码，只是前端测试。

# 感悟
真是越开发越知道测试的重要性，越测试越喜欢测试。不写测试代码用眼去开，那是懒惰的表现。而且测试代码也不需要多么DRY，能用就好。本来操作就是琐碎的。

不写测试，反而会浪费越来越多的时间。

# 参考
- http://edgeguides.rubyonrails.org/testing.html
- http://brandonhilkert.com/blog/7-reasons-why-im-sticking-with-minitest-and-fixtures-in-rails/
- http://api.rubyonrails.org/classes/ActiveRecord/FixtureSet.html
