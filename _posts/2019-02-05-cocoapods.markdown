---
layout: post
title:  "[iOS] Swift버전 변경 시 pod프로젝트 빌드 안되는 이슈 해결방법"
date:   2019-02-05 23:10:00 +0900
categories: iOS
---
Swift언어의 버전은 금방 업데이트가 되나 우리가 사용하는 라이브러리들은 언어에 대한 대응이 늦을 때가 많습니다.  
따라서 진행중인 프로젝트의 Swift버전을 올리게 되면 pod 프로젝트의 버전도 함께 올라가게 되어 빌드가 실패하는 경우가 종종 발생됩니다.

2019년 2월 5일 기준 Swift버전은 4.2까지 공개가 되었습니다.  
Swift5 공개도 얼마 안남았고, 제가 Swift4.0에서 4.2로 프로젝트 세팅을 변경했을때 발생한 이슈에 대해 해결방법을 제시하고자 합니다.


해결방법은 의외로 간단합니다.  
podfile 하단에 아래의 코드를 작성해서 pod 프로젝트의 Swift버전을 강제로 수정해줍니다.
{% highlight sh %}
post_install do |installer|
    installer.pods_project.targets.each do |target|
        if ['$lib_name'].include? target.name
            target.build_configurations.each do |config|
                config.build_settings['SWIFT_VERSION'] = '4.0'
            end
        end
    end
end
{% endhighlight %}

빌드 타겟이 문제가 될시는 다음 코드도 함께 삽입해줍니다.
{% highlight sh %}
    config.build_settings['IPHONEOS_DEPLOYMENT_TARGET'] = '9.0'
{% endhighlight %}
