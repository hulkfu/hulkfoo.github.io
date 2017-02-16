---
layout: post
title: Rails里MVC中VC的关系
---

在MVC里，Model算是比较独立的，要写成放在哪里都可以通用，View和Controller其实是比较交合的，C传参给V，V返回结果给C.

* C传参给V，靠的是实例变量的复制。
* V穿参给C，靠的是POST和GET等HTTP请求。

在Rails里，controller是在actionpack里定义的，view自然在actionview里定义。

# 实例变量是如何从Controller传到View里的

Rails中Controller里实例变量，可以直接在相应的View里使用，这一点很方便，像魔法。
而实现起来就是复制和粘贴。

下面以Rails 5的代码进行分析。

## 1. 在Controller里读取实例变量及其值，返回一个包含它们的Hash

```ruby
#actionpack/lib/abstract_controller/rendering.rb  line:67
def view_assigns
  protected_vars = _protected_ivars
  variables      = instance_variables

  # 除去保护的变量
  variables.reject! { |s| protected_vars.include? s }
  variables.each_with_object({}) { |name, hash|
    hash[name.slice(1, name.length)] = instance_variable_get(name)
  }
end
```

## 2. 生成ActiveView实例时将上面的 hash 赋值为其实例变量

```ruby
#actionview/lib/action_view/base.rb
def assign(new_assigns) # :nodoc:
  @_assigns = new_assigns.each { |key, value| instance_variable_set("@#{key}", value) }
end

def initialize(context = nil, assigns = {}, controller = nil, formats = nil) #:nodoc:
  @_config = ActiveSupport::InheritableOptions.new

  if context.is_a?(ActionView::Renderer)
    @view_renderer = context
  else
    lookup_context = context.is_a?(ActionView::LookupContext) ?
      context : ActionView::LookupContext.new(context)
    lookup_context.formats  = formats if formats
    lookup_context.prefixes = controller._prefixes if controller
    @view_renderer = ActionView::Renderer.new(lookup_context)
  end

  # 在这里赋值controller里的实例变量
  assign(assigns)
  assign_controller(controller)
  _prepare_context
end
```

而上面的初始化函数在 actionview/lib/action_view/rendering 里被使用：

```ruby
# An instance of a view class. The default view class is ActionView::Base.
#
# The view class must have the following methods:
# View.new[lookup_context, assigns, controller]
#   Create a new ActionView instance for a controller and we can also pass the arguments.
# View#render(option)
#   Returns String with the rendered template
#
# Override this method in a module to change the default behavior.
def view_context
  view_context_class.new(view_renderer, view_assigns, self)
end
```

```ruby
# Find and render a template based on the options given.
# :api: private
def _render_template(options)
  variant = options.delete(:variant)
  assigns = options.delete(:assigns)
  context = view_context

  context.assign assigns if assigns
  lookup_context.rendered_format = nil if options[:formats]
  lookup_context.variants = variant if variant

  view_renderer.render(context, options)
end
```

```ruby
def render_to_body(options = {})
  _process_options(options)
  _render_template(options)
end
```

## 3. 上面ActionView::Base在ActionController里用render方法生成实例

在Controller里最后render相应的View。

```ruby
# actionpack/lib/abstract_controller/rendering.rb
def render(*args, &block)
  options = _normalize_render(*args, &block)
  rendered_body = render_to_body(options)
  if options[:html]
    _set_html_content_type
  else
    _set_rendered_content_type rendered_format
  end
  self.response_body = rendered_body
end

# Performs the actual template rendering.
# :api: public
def render_to_body(options = {})
end
```

上面的render方法里会调用定义在template的render_to_body方法，而template正是ActionView的实例。

render方法定义在Rendering模块里，继承自AbstractController的Metal类还没有include，到了
Metal的子类ActionController::Base类，才被include。Metal类只是实现了Rack的接口，Base才到了Rails
的世界，一下包含了很多模块。

```ruby
MODULES = [
    AbstractController::Rendering,
    AbstractController::Translation,
    AbstractController::AssetPaths,

    Helpers,
    UrlFor,
    Redirecting,
    ActionView::Layouts,
    Rendering,
    Renderers::All,
    ConditionalGet,
    EtagWithTemplateDigest,
    Caching,
    MimeResponds,
    ImplicitRender,
    StrongParameters,

    Cookies,
    Flash,
    FormBuilder,
    RequestForgeryProtection,
    ForceSSL,
    Streaming,
    DataStreaming,
    HttpAuthentication::Basic::ControllerMethods,
    HttpAuthentication::Digest::ControllerMethods,
    HttpAuthentication::Token::ControllerMethods,

    # Before callbacks should also be executed as early as possible, so
    # also include them at the bottom.
    AbstractController::Callbacks,

    # Append rescue at the bottom to wrap as much as possible.
    Rescue,

    # Add instrumentations hooks at the bottom, to ensure they instrument
    # all the methods properly.
    Instrumentation,

    # Params wrapper should come before instrumentation so they are
    # properly showed in logs
    ParamsWrapper
  ]

  MODULES.each do |mod|
    include mod
  end
  setup_renderer!
```


# 参考
* http://stackoverflow.com/questions/18855178/how-are-rails-instance-variables-passed-to-views
* http://fantaxy025025.iteye.com/blog/1478273
