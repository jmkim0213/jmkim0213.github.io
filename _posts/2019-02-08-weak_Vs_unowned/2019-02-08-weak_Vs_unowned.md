---
layout: post
title:  "[iOS] 강한참조와 약한참조 (strong vs weak VS unowned)"
date:   2019-02-08 19:30:00 +0900
categories: [iOS, Swift]
---

# 강한참조 vs 약한참조
ARC(Automatic Reference Counting)는 `컴파일 시점`에 retain relase 와 같은 메모리 관리 메소드를 자동으로 삽입해줍니다. 가비지 컬렉터(Garbage collection)와 달리 참조사이클을 자동으로 처리하지 않기 때문에, 객체에 대해 강한참조가 남아 있는한 해당 객체는 메모리 해제가 되지 않게 됩니다.

따라서, 서로 다른 객체가 서로를 강하게 참조하게 되면 `순환참조가 발생해 메모리 누수(Memory Leak)`가 발생됩니다. 아래의 코드는 Foo와 Bar가 서로를 강하게 참조하여 순환참조가 발생되는 코드입니다.

![](https://user-images.githubusercontent.com/4909483/52545320-e8f52f80-2df9-11e9-992b-66ddfcf3291b.jpg)

{% highlight swift %}
class Foo {
  var bar: Bar?
}

class Bar {
  var foo: Foo?
}

let foo = Foo()
let bar = Bar()

foo.bar = bar   // Foo가 Bar를 강하게 참조
bar.foo = foo   // Bar가 Foo를 강하게 참조
{% endhighlight %}

---

그럼 순환참조를 방지하려면 어떻게 해야할까요?  
바로 Foo나 Bar중 하나가 강한참조가 아닌 약한참조를 가지면 됩니다.  
약한참조는 `weak` 또는 `unowned` 키워드를 사용합니다.

`weak`와 `unowned` 키워드를 지정한 참조변수는 참조하는 객체에 대해서 참조계수를 증가시키지 않기 때문에 순환참조를 방지할 수 있습니다. (약한참조)

![](https://user-images.githubusercontent.com/4909483/52551571-62077d80-2e20-11e9-94f7-67bee59cdf3e.png)

{% highlight swift %}
class Foo {
    var bar: Bar?
}

class Bar {
    weak var foo: Foo?
}

let foo = Foo()
let bar = Bar()

foo.bar = bar     // Foo가 Bar를 강하게 참조
bar.foo = foo     // Bar가 Foo를 약하게 참조
{% endhighlight %}

---
# weak vs unowned
 weak와 unowned는 모두 약한참조를 위한 키워드로, 참조하는 객체의 참조계수에 영향을 주지 않습니다.  
 그럼 이 둘은 어떤 차이가 있을까요? 
 
weak키워드는 optional에 사용할 수 있으며, unowned는 non-optional인 경우에만 사용 가능합니다.  
따라서 unowned는 옵셔널 추출이 필요하지 않지만, 참조하는 객체의 메모리가 해제된 상태에서 객체에 메세지를 보내면 bad access에러가 발생됩니다.

---

**if. 참조하는 객체의 참조계수가 0이 되어 메모리가 해제되는 경우.**

 > weak는 참조 값이 nil로 대체됩니다.  
 unowned는 참조 값이 그대로 유지됩니다.

---
 
 weak와 unowned는 클로저에서의 메모리 누수를 방지 하기 위해 캡쳐리스트에서도 사용될 수 있습니다.

{% highlight swift %}
let unownedClosure = { [unowend self] in
  /*  옵셔널 추출이 필요치 않으나, 
      self의 메모리가 해제된 상태에서 'self.doProccess()'을 수행하게 되면 에러가 발생됩니다! */
  self.doProccess()
}   

let weakClosure = { [weak self] in
  // 옵셔널 추출과정이 필요하지만 더 안전합니다.
  guard let self = self else { return }
  self.doProccess()
}
{% endhighlight %}

--- 

> unowned는 해제된 객체에 접근이 가능해 에러를 발생 시킬 수 있으므로,  
약한참조가 필요한 경우 weak 키워드만을 사용하고, guard let(또는 if let) 구문을 통해 안전하게 옵셔널 추출하는 것을 권장합니다.

---