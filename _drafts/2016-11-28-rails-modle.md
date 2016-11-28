---
layout: post
title: Rails Model
---


# ActiveModel
ActiveModel是从ActiveRecord分离出来的，主要负责模型相关，而后者主要负责数据库等其它工作。

# 主要参数

- class_name: 对应的类名
- foreign_key: 关联使用的外键，默认是 #{class_name}_id。
- primary_key: 指定属性名，比如select where primary_key = id
- as: 别名


# Assocaiation

## polymorphic
主要解决一张表要记录多种类型的数据。这样方便筛选。

比如一张评论表，我想记录照片的评论，又想记录文章的评论，那么就用polymorphic吧，而不是建两张表。

polymorphic 翻译来就是“多态”，它是面向对象的基石。
它使程序能够把不同种类的东西当作相同的东西来处理，从而做到更高层的抽象。

所以它还能实现继承，让所有子类在一张表里。

实现上Java用接口，Ruby用了Duck Type。

那么在表数据中，如何来表示这条记录是哪个态呢？多加一个type字段，一般是xxxxable_type.

那如果这条数据和其它数据关联，如何记录？多加一个那个关联数据的id，一般是xxxxable_id.

这就是Rails里model里的polymorphic的功能。

具体例子参考[RailsCasts](1)。在这个例子中comment可以是Article、Photo和Event的comment。

它们model如下：

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

这里也可以理解为Comment通过：

```rb
belongs_to :commentable, :polymorphic => true
```

创建了commentable父类，下面的Article、Phote和Event里评论就是commentable的子类：

ArticleComment、PhoteComment和EventComment，只不过它们都存在comments这张表里，靠表里的
commentable_type属性来区分，同时通过commentable_id来记录是哪个article、phote或event的
comment。如果是article的，那么commentable_type就是"atricle"，commentable_id就是article
的id。

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
  params.each do |name, value|  # 查找表单里的内容
    if name =~ /(.+)_id$/ # 比如是，photo_id
      return $1.classify.constantize.find(value) # value 就是phote的id
    end
  end
  nil
end
```


在数据库migration里，可以用 t.references 来创建表结构：

```rb
class CreatePictures < ActiveRecord::Migration[5.0]
  def change
    create_table :comments do |t|
      t.string :content
      t.references :commentable, polymorphic: true, index: true
      t.timestamps
    end
  end
end
```

[1]: http://railscasts.com/episodes/154-polymorphic-association
[2]: http://guides.rubyonrails.org/association_basics.html#polymorphic-associations
