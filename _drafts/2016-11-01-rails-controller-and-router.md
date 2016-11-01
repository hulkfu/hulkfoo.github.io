---
layout: post
title: Rails里的Controller和Router
---

Router定义了外部访问时URL的样子，通过筛选后，交给对应的Controller处理。



# Router

## Resourceful Routes

## Non-Resourceful Routes

而往往我们构造的页面，不会像脚手架生成的那样死板。

- :controller 对应app里的一个Controller
- :action 对应Controller里的Action
- :其他 可以自定义，会传到params里
- () 表示这是可选的

比如：

```rb
get ':controller(/:action(/:id))'
```

- 路径： /photos/show/1?user_id=2
- 得到： params = { controller: 'photos', action: 'show', id: '1', user_id: '2' }

```rb
get ':controller/:action/:id/:user_id'
```

- 路径： /photos/show/1/2
- 得到： params = { controller: 'photos', action: 'show', id: '1', user_id: '2' }

```rb
get ':controller/:action/:id/with_user/:user_id'
```

- 路径： /photos/show/1/with_user/2
- 得到： params = { controller: 'photos', action: 'show', id: '1', user_id: '2' }




# Controller



## ActionController::Parameters

为了安全，一般不在Controller里直接使用params这个变量，而是通过permit筛选一遍后只提取许可的参数。
从而保证入口参数可控可靠。

- require 提取一个或多个key的值
- permit 授权一个多个参数

```rb
params = ActionController::Parameters.new(person: { name: 'Francesco' })

# require后还是一个Parameters，并且没有permitted，只是将person提取出来。
params.require(:person)
=> <ActionController::Parameters {"name"=>"Francesco"} permitted: false>

# 现在params又嵌了一层
params
=> <ActionController::Parameters {"person"=><ActionController::Parameters {"name"=>"Francesco"} permitted: false>} permitted: false>

# 只有使用permit后，才能permitted，然后可以使用
person_params = params.require(:person).permit(:name)
=> <ActionController::Parameters {"name"=>"Francesco"} permitted: true>

person_params[:name]
=> "Francesco"

# 或一开始就只用permit，得到一个hash参数
permitted_params = params.permit(person:[:name])
=> <ActionController::Parameters {"person"=><ActionController::Parameters {"name"=>"Francesco"} permitted: true>} permitted: true>

permitted_params[:person][:name]
=> "Francesco"
```

# 参考
- http://guides.rubyonrails.org/routing.html
- http://api.rubyonrails.org/classes/ActionController/Parameters.html
