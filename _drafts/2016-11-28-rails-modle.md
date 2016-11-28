---
layout: post
title: Rails Model
---

# Assocaiation

## polymorphic-association
多态是面向对象的基石，它使程序能够把不同种类的东西当作相同的东西来处理，从而做到更高层的抽象。

实现上Java用接口，Ruby用了Duck Type。

那么在表数据中，如何来表示这条记录是哪个态呢？多加一个type字段。

那如果这条数据和其它数据关联，如何记录？多加一个那个关联数据的id。

其实，这就是Rails里model里的polymorphic的功能。

具体例子参考[RailsCasts](1)。在这个例子中comment可以是Article、Photo和Event的comment。它们model如下：

```ruby
class Comment < ActiveRecord::Base
  belongs_to :commentable, :polymorphic => true
end

class Article < ActiveRecord::Base
  has_many :comments, :as => :commentable
end

class Photo < ActiveRecord::Base
  has_many :comments, :as => :commentable
  #...
end

class Event < ActiveRecord::Base
  has_many :comments, :as => :commentable
end
```

使用belongs_to和has_many这么“通俗”的方法就搞定了，关键是下面的参数。

belongs_to后面的polymorphic为true，has_many后的as为那个多态的东西。

可以想象，当rails发现一个modle的belongs_to后的polymorphic为true时，它就会把前面的多态东西commentable记下来，生成动态方法，当生成数据时就会记录下commentable_type和commentable_id。

has_many中的as指明了comments的类别，这样就能有相应的动态方法了。

comments的controller如下：

```ruby
def index
  @commentable = find_commentable
  @comments = @commentable.comments
end

def create
  @commentable = find_commentable
  @comment = @commentable.comments.build(params[:comment])
  if @comment.save
    flash[:notice] = "Successfully created comment."
    redirect_to :id => nil
  else
    render :action => 'new'
  end
end

private

def find_commentable
  params.each do |name, value|
    if name =~ /(.+)_id$/
      return $1.classify.constantize.find(value)
    end
  end
  nil
end
```

其中的find_commentable方法值得参考，匹配/(.+)_id$/，得到commentable的name即其类，然后根据其value也就是id找到这个commentable。

[1]: http://railscasts.com/episodes/154-polymorphic-association
[2]: http://guides.rubyonrails.org/association_basics.html#polymorphic-associations
