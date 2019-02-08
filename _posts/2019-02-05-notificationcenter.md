---
layout: post
title:  "[iOS] NotificationCenter 가독성 높이기 (with RxSwift + Protocol)"
date:   2019-02-05 21:32:58 +0900
categories: [iOS, Swift, RxSwift]
---
NotificationCenter는 전역적인 이벤트를 처리하기 위해 사용되는 유용한 클래스입니다.  
그러나 NotificationName을 제대로 정리하지 않으면, 가독성과 유지보수가 어려워지는 문제점이 있습니다.  
이번 포스트에서는 Protocol + Enum + RxSwift를 활용해서 NotificationCenter를 쉽게 사용하는 방법을 소개해보자 합니다 :)

- 일단 NotificationCenterProtocol를 작성합니다.  
Notification.Name을 반환하는 프로퍼티를 하나 선언한 뒤에 옵저버 등록과 이벤트 post 기능을 구현합니다.

{% highlight swift %}
import Foundation
import RxSwift

protocol NotificationCenterProtocol {
    var name: Notification.Name { get }
}

extension NotificationCenterProtocol {
    func addObserver() -> Observable<Any?> {
        return NotificationCenter.default.rx.notification(self.name).map { $0.object }
    }
    
    func post(object: Any? = nil) {
        NotificationCenter.default.post(name: self.name, object: object, userInfo: nil)
    }
}
{% endhighlight %}



- 위와 같이 기본 프로토콜을 정의 한 후에 Enum 타입으로 프로토콜을 구현해줍니다.  
여기서 Custom Notification Name이 겹치지 않게 프로토콜을 구현한 Enum타입의 이름을 네임스페이스로 활용합니다.

{% highlight swift %}
import Foundation
import UIKit

enum ApplicationNotificationCenter: NotificationCenterProtocol {
    case willEnterForeground
    case didReceiveMemoryWarning
    case custom

    var name: Notification.Name {
        switch self {
        case .willEnterForeground:
            return UIApplication.willEnterForegroundNotification
        case .didReceiveMemoryWarning:
            return UIApplication.didReceiveMemoryWarningNotification
        case .custom:
            return Notification.Name("ApplicationNotificationCenter.custom")
        }
    }
}
{% endhighlight %} 

- Enum타입을 구현했으면 다음과 같이 이벤트를 옵저빙 할 수 있게 됩니다.

{% highlight swift %}
ApplicationNotificationCenter.custom.addObserver().bind { object in
    guard let object = object as? String else { return }
    print(object)
}.disposed(by: self.disposeBag)
{% endhighlight %}



- 이벤트 발생 시점에 다음과 같이 작성하면 NotificationCenter를 통해 이벤트를 Post할 수 있게 됩니다.

{% highlight swift %}
ApplicationNotificationCenter.custom.post(object: "custom event발생!")
{% endhighlight %}