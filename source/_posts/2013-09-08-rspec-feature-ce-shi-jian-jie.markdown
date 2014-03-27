---
layout: post
title: "rspec中使用feature spec进行用户/验收测试"
date: 2013-09-08 15:55
comments: true
categories: Ruby
tags: [测试, capybara, rspec, ruby]
description: rspec和capybara在ruby程序员中很多人都不陌生了.在2.0版本以后的capybara中,新加入了feature spec的写法.在rspec中默认使用spec/feature,而不再使用spec/request.

---

##feature spec与request spec的区别
rspec和capybara在ruby程序员中很多人都不陌生了.在2.0版本以后的capybara中,新加入了feature spec的写法.在rspec中默认使用spec/feature,而不再使用spec/request.

feature spec在测试中来说是比较高等级的测试.大概相当于集成测试或者更高级别的用户/验收测试.一般是模拟用户的实际操作和应用外部的请求来执行测试.简单来说就是模拟浏览器中的动作,期待应用正确的反应.

为什么2.0之后的capybara要做出这样的改动呢?

因为一般的来说capybara是模拟浏览器中的动作,这种测试是相当于模拟一个用户在用户界面中的操作,是基于usercase的.而以前的request spec则还是没有脱离应用层面(发送http请求,比较相应结果),比如在以前的capybara的dsl中,使用'get'来访问页面,而现在则改成了'visit'.后者'访问'这个动作,比'get'这个http请求更接近于用户的行为.

所以使用feature spec相对来说更符合behavior的思想,也带来更好的阅读性.

##rails中使用feature spec
本文以rails为例,来介绍feature spec的使用.

首先安装rspec和capybara,在gemfile中加入

```ruby
group :development, :test do
  gem 'rspec-rails'
end

group :test do
  gem 'capybara'
end
```
之后执行

```
bundle install
```

再之后执行

```
rails generate rspec:install
```

rspec和capybara就安装完毕了.

rails中的generator默认生成的集成测试会在request/xxxx_spec.rb.默认是不支持capybara的,我们如果要在其中使用capybara还要在spec_helper中的configure中加入`config.include Capybara::DSL`

而我们要使用的feature/xxxx_spec.rb目前尚无法由rails自动生成,但是不用做任何设置就可以支持capybara的DSL.

所以我们手动建立spec/feature文件夹,在其中手动创建xxx_spec.rb的测试文件.

##feature spec写法
在feature spec中,我们使用`feature-scenario` 来替代之前rspec的`describe - it`模式
注意两者千万不要混写成`feature-it`或者`describe-scenario`这种不伦不类的风格..

另外与describe不同,feature下面是不允许再套feature的.

```ruby
describe 'Static pages' do
  describe 'Home page' do
    it "should have the content 'Sample App'" do
      visit '/static_pages/home'
      expect(page).to have_content('Sample App')
    end
  end
end
```

这样是可以的,但是把describe换成feature则会报错.

feature spec的测试中,我们一般按照待测的功能点来划分用例,即每一个feature.而每一个scenario对应一条user case.这一点和测试人员写测试用例的方法很相似.比如下面的2条测试用例

|功能点|用户输入|期望结果|
|-----|-------|:------:|
|用户登录| 用户输入正确的用户名和密码|登录成功|
|| 用户输入错误的用户名和密码|登录失败|


使用feature spec的方式写出的测试代码如下.

```ruby
require "spec_helper"

feature 'Login' do
  scenario 'User enter right account' do
    visit '/login'
    fill_in 'Name', :with => 'username'
    fill_in 'Password', :with => 'right'
    click_button 'login'

    expect(page).to have_text('welcome back,username')
  end

  scenario 'User enter wrong account' do
    visit '/login'
    fill_in 'Name', with: 'username'
    fill_in 'Password', with: 'wrong'
    click_button 'login'

    expect(page).to have_text('Wrong password,please try again!')
  end
end
```

feature spec的简单介绍就写到这里,希望大家喜欢这种方式.



