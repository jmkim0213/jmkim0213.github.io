---
layout: post
title:  "[Swift] class와 struct 그리고 Copy On Write"
date:   2019-02-18 21:30:00 +0900
categories: [iOS, Swift]
---

### 서론
Swift로 데이터의 집합을 표현하는 방법으로는 struct, class, tuple, array등 다양합니다.
오늘은 이 중에서 주로 데이터 모델을 작성할때 사용하는 struct와 class에 대해 알아 보고자 합니다.

class와 struct는 둘 다 Property와 Function을 가지고 있으며, 프로토콜을 구현할 수 있는 공통점이 있는 타입입니다. 
하지만 근본적으로 두 타입은 많이 다릅니다. 

---

# Class
class는 참조타입(Reference Type)으로 Heap Memory 공간에 할당되며, 메모리에 할당된 class는 인스턴스 또는 객체라고 불리고 있습니다. 그리고 그 인스턴스를 참조(또는 가리키는)하는 변수를 참조변수라고 합니다.

---

> 참조변수는 대입연산자(=)사용시 참조하는 인스턴스의 참조 값을 복사합니다.

---

{% highlight swift %}
class Foo {           // class
  var data: Int = 0   // property
}
let foo1 = Foo()      // 새로운 instance 생성 후 참조변수 foo1가 참조하도록 참조 값 대입
let foo2 = foo1       // foo1의 참조 값을 foo2에 대입
{% endhighlight %}

![](https://user-images.githubusercontent.com/4909483/52785580-a783af00-309b-11e9-9c8d-cf8eadc32554.png)

---

# Struct
struct는 값 타입(Value Type)으로 Stack, Heap, Data Memory 공간에 할당될 수 있습니다.

---

> Struct타입의 변수는 대입연산자(=)사용시 소유한 모든 Property의 값을 복사합니다. 

---

{% highlight swift %}
struct Foo {          // struct
  var data: Int = 0   // property
}
let foo1 = Foo()      // 새로운 구조체(foo1) 생성
let foo2 = foo1       // 새로운 구조체(foo2) 생성 후 foo1의 값을 대입
{% endhighlight %}

![](https://user-images.githubusercontent.com/4909483/52786090-4230bd80-309d-11e9-8b2f-ffa1fa3c3bff.png)

---

# 데이터 모델에서의 class
참조타입인 class를 데이터 모델로 사용한다면, 한 인스턴스에 대해 많은 참조가 존재하게되어 인스턴스의 데이터 변경시 참조하는 모든 곳이 영향을 받아 의도치 않은 문제가 발생 할 수 있습니다.

좀 더 풀어서 이야기해보겠습니다.   
Foo와 Bar가 동일한 인스턴스 fooBar를 참조하고 있고, Foo의 어느 함수 내에서 fooBar 상태를 변경했다고 가정해봅시다. Bar도 fooBar를 참조해서 사용하고 있기때문에, Foo에서 변경된 fooBar상태 변경으로 인해 Bar에서 의도치 않은 문제가 발생 할 수도 있습니다.

또한, 이렇게 발생된 문제는 참조가 많은 경우에 문제의 발생지를 찾기 어려워져 디버깅을 하기가 쉽지 않습니다.

--- 

# 데이터 모델에서의 struct
값 타입인 struct를 데이터 모델로 사용한다면, 위에서 거론한 class의 문제 점을 해결 할 수 있습니다.
struct는 대입시 참조가 아닌 값을 복사하기 때문에, struct의 값을 변경한다해도 그 값이 다른 곳에 영향을 주지 않기 때문입니다.

하지만 struct의 덩치가 커지게되면 struct간 대입연산(=)시에 모든 값이 복사되어야 하기 때문에 속도적인 면에서 큰 비용이 발생합니다. 또한 struct의 값이 변경될 필요가 없음에도 불구하고 대입연산시 매번 새로운 메모리 공간을 할당해 프로퍼티를 복사하므로 불필요한 메모리 할당이되는 문제가 발생합니다.

위 문제는 Copy on write라는 개념을 도입해 해결이 가능합니다.

---

### Copy on write
Copy on write (이하 COW)는 데이터 복사시 실제로 값을 복사되지 않고, 동일한 값을 참조하다가 데이터 변경이 발생될 시에 복사해 값을 변경하는 기법입니다.

Swift에서는 원시타입 구조체(Int, Double, String)와 Array, Set, Dictionary등 컬렉션 구조체에 이미 구현되어있습니다. 

{% highlight swift %}
var arr1 = [0, 1, 2]    // [Int]배열이 생성됩니다.
var arr2 = arr1         // 이 경우 실제 복사가 일어나지 않고, 동일한 주소를 가지게 됩니다.
arr1.append(3)          // 이때 [0, 1, 2]배열이 복사가 되며, arr1은 [0, 1, 2, 3]의 값을 가지게 되고 arr2와 다른 주소를 가지게 됩니다.
{% endhighlight %}

---

# 사용자 정의 구조체에서의 COW
사용자 정의 구조체에는 COW가 구현되어있지 않기 때문에 필요하다면 구현해줘야 합니다.  
주석으로 간단한 예제를 설명하면서 COW를 구현해보도록 하겠습니다.

{% highlight swift %}
// 일단 사용자 정의 구조체 UserData를 정의합니다.
struct UserData {
    var intValue: Int       = 0
    var strValue: String    = ""
}

/*
대입연산에 의해 즉각적으로 프로퍼티 복사가 발생하는 것을 막기 위해 
참조타입인 Class를 이용해 UserData를 한번 wrapping 해줄 필요가 있습니다.
또한, UserData말고도 다른 타입에 대해 범용적으로 활용할 수 있도록 
Generic(<T>)으로 데이터타입에 대한 유연성을 부여해줍니다.
*/
struct DataWrapper<T> {
    var data: T

    init(data: T) {
      self.data = data
    }
}

/*
  DataWrapper를 제어해줄 CowData 구조체를 선언합니다.
*/
struct CowData<T> {
    // Data Wrapper
    private var dataWrapper: DataWrapper<T>
    init(data: T) {
        self.dataWrapper = DataWrapper(data: data)
    }
    
    var data: T {
        get {
            return self.dataWrapper.data
        }
        set {
            if !isKnownUniquelyReferenced(&self.dataWrapper) {
                // dataWrapper에 대한 참조가 Uniquely하지 않으면 새로운 Wrapper를 생성하여 값을 대입해줍니다.
                self.dataWrapper = DataWrapper(data: newValue)
            } else {
                // dataWrapper에 대한 참조가 Uniquely하다면 기존 Wrapper에 대해서 struct 값을 대입해줍니다.
                self.dataWrapper.data = newValue
            }
        }
    }
}

var cowData1 = CowData(data: UserData())
cowData1.data.strValue = "i'm UserData1"

// cowData2의 dataWrapper는 cowData1와 동일한 참조를 가지게 됩니다.
var cowData2 = cowData1                  

print("!! cowData2의 dataWrapper는 cowData1와 동일한 참조를 가지고 있습니다.")
print("cowData1.data.strValue: \(cowData1.data.strValue)")
print("cowData2.data.strValue: \(cowData2.data.strValue)\n")

 // cowData2의 dataWrapper가 새로운 struct값과 함께 새롭게 할당됩니다.
cowData2.data.strValue = "i'm UserData2"       

print("!! cowData2의 dataWrapper는 cowData1와 다른 참조를 가지고 있습니다.")
print("cowData1.data.strValue: \(cowData1.data.strValue)")
print("cowData2.data.strValue: \(cowData2.data.strValue)\n")
{% endhighlight %}

---

# 결과화면
![](https://user-images.githubusercontent.com/4909483/52954611-f8621300-33cd-11e9-94e6-f7d9445764d3.png)

---

참고: [https://docs.swift.org/swift-book/LanguageGuide/ClassesAndStructures.html](https://docs.swift.org/swift-book/LanguageGuide/ClassesAndStructures.html)

---