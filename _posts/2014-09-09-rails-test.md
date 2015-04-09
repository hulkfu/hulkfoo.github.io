---
layout: post
title: Rails Test
---

把经常在做的测试过程抽象成测试用例代码，比如功能函数、操作过程、性能等任何能想到的。

# RSpec
在Rails下需要使用[rspec-rails](https://github.com/rspec/rspec-rails)，其官网参考文档很详细。

## 注意
* 把.rspec里的--warning去掉，否则会有一大堆的警告。

# [FactoryGirl](https://github.com/thoughtbot/factory_girl)
如其名，生成model实例的。

如果使用rails，需要用factory_girl_rails这个gem。[这里有入门](https://github.com/thoughtbot/factory_girl/blob/master/GETTING_STARTED.md)。

使用前可以在rspec里配置includeFactoryGirl的DSL，这样就不需要在测试时前面每次都加FactoryGirl了。

```
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
No matter which strategy is used, it's possible to override the defined attributes by passing a hash:

# Build a User instance and override the first_name property
user = build(:user, first_name: "Joe")
user.first_name
# => "Joe"
```

FactoryGirl还有继承、关联、序列等用法，可以参考上面提到的文档。

## 其它
当使用carriewave时，直接在factories文件里用**File.open**打开文件。如：

```
FactoryGirl.define do
  factory :task do
    name "task"
    attachment File.open(File.join(Rails.root, 'spec', 'support', 'attachments', 'file.txt'))
  end
``

# [capybara](https://github.com/jnicklas/capybara)
它模仿浏览器的行为，所以使用它就像自己在操作浏览器一样。

也因此只有在feature测试中才能使用。

有时capybara提示ElementNotFound，可能是真的没有找到，也可能是其它地方出了问题，然后它只catch到这个错误。此时去浏览器里重现一下场景看看。

# 感悟
真是越开发越知道测试的重要性，越测试越喜欢测试。不写测试代码用眼去开，那是懒惰的表现。而且测试代码也不需要多么DRY，能用就好。本来操作就是琐碎的。

不写测试，反而会浪费越来越多的时间。