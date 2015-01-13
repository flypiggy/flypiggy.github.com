title: 一个简单的评论过滤系统的实现
date: 2015-01-13 23:28:32
categories: Rails
tags: [rails, ruby, 垃圾评论]
description: 网站开启匿名评论之后, 垃圾评论就一直是一个让人头疼的问题. 关键词过滤及限制评论数很容易被找到针对的办法, 上 NLP 又太重. 最后使用了语句的相似度检查来完成一个简单的评论过滤系统.
---

最近公园网站开启匿名评论之后, 垃圾评论就一直是一个让人头疼的问题. 关键词过滤及限制评论数很容易被找到针对的办法, 发垃圾评论的人会不停的通过空格, 符号等来绕过关键词的过滤. 对于一个每天评论不过百的网站来说, 上 [NLP](http://en.wikipedia.org/wiki/Natural_language_processing) 又太重了(其实是不懂).

##通过语句相似度过滤垃圾评论
后来我想到公园网站的垃圾评论大部分内容都是几个垃圾评论的变种, 内容的相似度都比较高. 所以我只需要维护一个包含这几个内容的列表, 并计算语句的相似度, 屏蔽掉相似度较高的评论就可以暂时达到过滤垃圾评论的目的.

研究如何计算过程中我找到了2种方式, [Levenshtein distance](http://en.wikipedia.org/wiki/Levenshtein_distance) 以及 [How to Strike a Match](http://www.catalysoft.com/articles/StrikeAMatch.html) . 最后我使用了后者计算相似度的方式实现了一个简单的评论过滤系统.

##代码实现

[text](https://github.com/threedaymonk/text)这个gem 同时提供了以上两种算法. 有兴趣的可以看一下源码中两种算法的具体实现.

Comment 中验证评论是否合法

```ruby
class Comment < ActiveRecord::Base
  ...
  validate :check_spam
  ...
 
  def check_spam
    #遍历检查 spam 中条目与评论的相似度, 如果大于设定比例则添加 error, 验证失败
    Spam.all.each do |spam|
      if Text::WhiteSimilarity.new.similarity(body, spam.body) > (spam.similarity || 0.9)
        errors.add(:body, "内容不合法")
        return
      end
    end
    # 对于 spam 中没有收录的模板或漏网之鱼检查评论与最近的3条评论的相似度
    # 如果相似度大于95%则过滤, 一般垃圾评论都是连续刷评论的.
    Comment.last(3).each do |comment|
      if Text::WhiteSimilarity.new.similarity(body, comment.body) > 0.95
        errors.add(:body, "内容不合法")
        return
      end
    end
  end  
end

```

加上这层防护之后, 应该可以暂时安心了.