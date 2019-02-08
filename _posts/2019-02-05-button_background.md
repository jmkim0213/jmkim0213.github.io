---
layout: post
title:  "[iOS] state 별로 UIButton의 백그라운드 컬러변경하는 방법"
date:   2019-02-05 23:50:00 +0900
categories: [iOS, Swift, UI]
---
UIButton 컴포넌트에는 normal, highlighted, disabled, selected 등 여러 상태에 따른 이미지를 지정할 수 있습니다.  
그런데 왜 그런지는 모르겠지만 당연하게 있어야할거 같은 backgroundColor를 각 상태에 맞게 지정하는 방법을 지원하지 않습니다.

이에 약간의 꼼수를 부려 BackgroundColor를 상태 별로 지정하는 코드를 소개해보자 합니다.

{% highlight swift %}
import UIKit

extension UIButton {
    func setBackgroundColor(_ color: UIColor, for state: UIControl.State) {
        UIGraphicsBeginImageContext(CGSize(width: 1.0, height: 1.0))
        guard let context = UIGraphicsGetCurrentContext() else { return }
        context.setFillColor(color.cgColor)
        context.fill(CGRect(x: 0.0, y: 0.0, width: 1.0, height: 1.0))
        
        let backgroundImage = UIGraphicsGetImageFromCurrentImageContext()
        UIGraphicsEndImageContext()
         
        self.setBackgroundImage(backgroundImage, for: state)
    }
}
{% endhighlight %}

코드는 간단합니다. ImageContext를 생성하여 색으로 채운 뒤 이미지를 생성합니다.  
그리고 생성된 이미지를 setBackgroundImage함수를 이용해 각 상태별로 지정합니다.

위 코드 작성이 끝나면 아래와 같이 사용하면 됩니다.
{% highlight swift %}
let button = UIButton()
button.setBackgroundColor(.red, for: .normal)
button.setBackgroundColor(.blue, for: .selected)
button.setBackgroundColor(.gray, for: .disabled)
{% endhighlight %}
