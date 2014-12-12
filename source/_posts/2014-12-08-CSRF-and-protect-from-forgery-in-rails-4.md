title: CSRF and protect_from_forgery in Rails 4
date: 2014-12-08 00:20:31
categories: Rails
tags: [rails, ruby]
description: 最近的一个项目频繁出现恼人的 InvalidAuthenticityToken 错误, 引发了我对 Rails 中 protect_from_forgery 的思考, 本文介绍 CSRF 攻击以及在 Rails 中的保护方式, 什么情况下应该使用那种策略来防CSRF.

---

最近的一个项目[2014中国互联网创新评选](http://innoawards.geekpark.net/)频繁出现恼人的 InvalidAuthenticityToken 错误, 引发了我对 Rails 中 protect_from_forgery 的思考, 本文介绍 CSRF 攻击以及在 Rails 中的保护方式, 什么情况下应该使用那种策略来防CSRF.

##CSRF

CSRF（Cross-site request forgery) 跨站请求伪造, 是一种比较常见的入侵方式. 顾名思义, 就是伪造成某个受信任的用户去向网站发送请求, 来达到某些目的. 比如伪造成你向网站提交一个修改密码的表单, 将你的密码改为攻击者设定的某个密码, 这样在不知不觉中你的密码就被修改了, 而假设攻击者同时也知道你的用户名, 这样攻击者盗取了你的帐号. 

CSRF 一般是怎么发生的呢? 

一般我们登录网站后, 都会和网站建立起一个一对一的 session 或 cookie 联系. 你接下来所发出的请求, 会带有你特有的一个身份信息, 服务器会识别出哪个用户执行的操作, 这样接下来你进行其它的操作时, 比如发帖, 转账等就不需要再次进行登录. 而 CSRF 所做的, 就是**通过你的浏览器**, 利用你和服务器间这特有的识别信息, 来通过你向服务器发出某个请求. 这样服务器认为这个操作是由你发出的, 从而"遵循你的意志", 执行接到的命令.

比如在 A 网站有一个表单, 你已经登录了 A 网站, 我在我的 B 网站写了一段 js 代码, 内容就是按照 A 网站的表单的格式向 A 网站提交一份表单, 表单的 value 是我自己定义的. 然后我只需要引导你访问 B 网站, 在你访问 B 网站期间, 这段 js 代码被你的浏览器所执行, 然后向 A 网站提交了表单, 这样我就成功利用了你的身份你的浏览器向 A 网站发出了请求. 而由于请求是由你的浏览器发出,可以利用你本地的 cookie 和 session 信息, 使其看起来就好像你发出的, 所以 A 网站的服务器也认为这个请求是来自你的合法请求. 这样不知不觉间, 你就执行了一个并非你本意的操作. 而一次 CSRF 攻击就成功了.

这种攻击原理简单, 实现也不难, 但是绝大多数 web 框架都已经对这种攻击有所防范, 但遗憾的是互联网上依然有很多网站对这种攻击毫无防范.

##Rails 中对 CSRF 的防范

Rails 中会在 html 页面生成时给每个表单添加一个隐藏的 authenticate_token 属性, 这个 token 在你每次登录网站时生成, 过期时间于 session 相同. 在你每次提交一个表单时都要带着这个 token, 并在 controller 中, 首先通过 `protect_from_forgery` 这个方法对 token 进行验证, 如果 token 验证失败的话, 你所执行的 post 操作就会失败. 

这样攻击者因为无法拿到你的 token, 所以就无法伪造整个表单. 前面所说的攻击方式中所提交的表单因为 token 错误, 而无法继续执行, 这样就达到了防范 CSRF 攻击的目的. 我们一般在 rails 的 layout 中会看到这样的代码.
```slim
doctype html
html lang=session[:locale]
  head
    title
      = (content_for?(:title) ? "#{yield(:title)} | " : '') + t(:sitename)
	...
    = csrf_meta_tags
    ...
  body class=(controller_name + ' ' + controller_name + '_' + action_name)
    = render 'share/header'
    = yield
    = render 'share/footer'
    = javascript_include_tag 'application'
```

其中`csrf_meta_tags`所做的就是生成这个 token
```html
<meta content="authenticity_token" name="csrf-param">
<meta content="c2tB7nB1S0s7nQD2++sjv3pT/bKtb+4GIrEfcnn/tWw=" name="csrf-token">
```
rails 会默认在表单中使用这个 token参数和 form 一起传过来.

##`protect_from_forgery`
token 传过来之后, rails 的 controller 会对这个 token 的合法性进行验证
```ruby
class ApplicationController < ActionController::Base
  # Prevent CSRF attacks by raising an exception.
  # For APIs, you may want to use :null_session instead.
  protect_from_forgery
end
```
在 controller 中, `protect_from_forgery` 所做的就是验证这个 token 是否正确, 可以在后面加入`:with`参数指定处理方式, 如`protect_from_forgery with: :null_session
`, rails 提供了三种处理方式

###`:exception`
这个是 rails4 的默认处理方式, 在 token 验证失败时抛出一个 InvalidAuthenticityToken 异常, 如果没有捕获的话进程终止, 客户端会收到服务器返回的 500错误. 这样所提交的伪造请求就无法继续执行到具体业务步骤, 达到了防范的目的.

###`:null_session`
这曾是 rails3 的默认处理方式, token 验证失败时并不抛出异常, 而是使用一个空的 session 来执行下面的业务处理. 因为我们一般会把大部分的用户识别信息都保存在 session 中, 而在业务执行时也会在 session 中查找相关的用户信息. 所以当使用一个空 session 来提交表单时候, 应用因为从 session 中无法找到用户的相关信息, 从而无法伪造成该用户来进行操作.

###`:reset_session`
完全的重置这个 session. 这个不怎么用到, 虽然也可以达到防范的目的, 但是用户的 session 会被完全清空, 会需要重新进行登录, 一般使用 :null_session 就够了.

###另外还有一个`skip_before_action :verify_authenticity_token`
以上的3种策略之外, 还有一种情况, 就是完全跳过对 csrf_token 的验证. 也就是放弃对 csrf 的保护. 在你完全知道自己在做什么的情况下, 慎用!

##什么情况下应该选用那种策略来应对

知道了有以上的几种策略, 就需要考虑什么时候适合用什么策略来应对. 其实这才是写本文的初衷, 前面废话比较多, 其实对 rails 和 csrf 有所了解的人直接跳到这里看就够了, 2333...

###`with: :exception`
如开始所说的, 在这个项目中 ([2014中国互联网创新评选](http://innoawards.geekpark.net/vote)), 一开始被`InvalidAuthenticityToken` 频繁骚扰让我很痛苦, 差点想把 csrf 改成 `:null_session` 方式. 但是在这个项目的评选中, 观众是完全匿名的, 我们只针对投票的 IP 地址的投票频率进行了限制, 如果使用`:null_session`的方式, 即使空的 session 也依然是可以投票的. 这样 csrf 的防范作用就没有了. 

厂商在拉票时一般都会推送一篇文章给粉丝们号召他们去投票, 如果没有做验证的话, 厂商只需要在推送的文章甚至在自己的主页中加入一段提交这个投票表单的 js 代码既可在浏览这个网页的用户不知情的情况下进行投票. 因为票会从用户的浏览器投出, 也就做到了 '分布式投票' 那么针对 IP 地址的限制就几乎没有什么作用了. 也就无法杜绝刷票现象.

所以最终这个项目我没有更改 csrf 的防护方式, 而是在代码中对`InvalidAuthenticityToken` 进行了处理
```ruby
rescue_from ActionController::InvalidAuthenticityToken do |exception|
  render_422(exception)
end
```

###`with: :null_session`
在做上一个项目的同时, 手中还有另一个项目([2015极客公园创新大会](http://gif.geekpark.net/cn))
在这个项目中, 只有一个下订单购票的表单, 用户同样是不需要登录就能提交的. 正常的用户一般不会直接进行购票, 而是浏览页面, 查看大会的特色, 嘉宾等等.  当他们觉得足够有吸引力, 终于下定决心要购票的时候, 页面生成的 session 以及 token 可能已经过期了. 这个使用如果依然使用`exception`的方式进行处理就会提示错误, 需要用户重新刷新页面后再次提交订单. 对用户下订单的流畅度造成了影响, 体验下降. 而且以买票来说, 未付款的订单是无效的, 不像投票一样会影响计票准确性.

所以最终在这个项目中我使用了`:null_session`的方式.

另外, 在 rails 中的一些 api 接口, 由于验证是通过其它方式进行的, rails 推荐的也是使用`:null_session`这种方式.

###`:reset_session`
这个用的比较少, 使用情况参照`:null_session`