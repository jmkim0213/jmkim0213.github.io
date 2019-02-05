---
layout: post
title:  "[iOS] RxSwift + NotificationCenter"
date:   2019-02-05 21:32:58 +0900
categories: iOS
---
NotificationCenter는 전역적인 이벤트를 처리하기 위해 사용되는 유용한 클래스이다.

그러나 NotificationName을 제대로 정리하지 않으면, 가독성과 유지보수가 어려워지는 문제점이 있다.

이번 포스트에서는 Protocol + Enum + RxSwift를 활용해서 NotificationCenter를 쉽게 사용하는 방법을 소개해보자 한다.


{% highlight swift %}
def print_hi(name)
  puts "Hi, #{name}"
end
print_hi('Tom')
#=> prints 'Hi, Tom' to STDOUT.
{% endhighlight %}
