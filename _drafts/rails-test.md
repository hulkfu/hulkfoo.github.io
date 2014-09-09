---
layout: post
title: Rails Test
---

# RSpec
在Rails下需要使用[rspec-rails](https://github.com/rspec/rspec-rails)，其官网参考文档很详细。

## 注意
* 把.rspec里的--warning去掉，否则会有一大堆的警告。

# [FactoryGirl](https://github.com/thoughtbot/factory_girl)
如其名，生成model实例的。

如果使用rails，需要用factory_girl_rails这个gem。[这里有入门](https://github.com/thoughtbot/factory_girl/blob/master/GETTING_STARTED.md)。

使用前可以配置test suite，这样就不需要在测试时前面每次都加FactoryGirl了。

```
# RSpec
# spec/support/factory_girl.rb
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


# [capybara](https://github.com/jnicklas/capybara)
它模仿浏览器的行为，所以使用它就像自己在操作浏览器一样。

也因此只有在feature测试中才能使用。

#

