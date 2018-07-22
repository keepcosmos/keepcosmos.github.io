---

layout: post
title: 저 새를 오리라고 부르겠다. 덕 타이핑!
category: "dev"
description: "Duck Typing에 대한 생각"
modified: 2015-03-01
comments: true
tags: [Duck typing, Dynamic type, 오리타입, 동적타입]
share: false

---

> When I see a bird that walks like a duck and swims like a duck and quacks like a duck, I call that bird a duck.

Java로 개발을 해오다 Ruby를 처음 시작했을 때의 느낌은 불안함이었다. 사고가 나도 누군가 지켜주지 못할 것 같다는 느낌은 [type safety](http://en.wikipedia.org/wiki/Type_safety)가 없었기 때문이다. Ruby 코드를 실행시켰을 때 종종 튕겨나왔던 `NoMethodError: undefined method 'xxx'`라는 메시지가 나를 더욱 불안하게 만들었다. [^undefined_method]  

Ruby에 익숙해지기까지 type(파라미터에 정의하지 않는)을 머릿속으로 그리며 스스로 type safety를 만들었다. 스스로 type safety를 만든다는 말이 우습지만, 문서나 내부 구현을 직접 보고 type을 유추하지 않으면 코드 한 줄도 머뭇거려졌다. 지금은 그 강박에서 자유롭다. 단지 익숙해졌기 때문이 아니라 코드를 쓸 때 생각하는 프레임이 변화한 것이다. 그것을 깨달은 것은 최근 Swift를 공부하기 시작하면서이다.  

몇 개의 Swift 코드를 써내려갔다. 당연히 Ruby 코드를 쓸 때 보다 안전하다는 느낌을 받는다던지 반대로 번거롭거나 귀찮은 느낌을 받아야 하겠지만, 나는 답답한 느낌뿐이었다. 사고가 제한된다는 느낌이다. Swift를 공부하기 전 까지 지난 2년간 Ruby, Javascript, R만 사용했다. 단지 언어의 낯설음 때문은 아니다.  

많은 사람들이 이야기하는 Dynamic type의 장점은 단순하고 짧은 코드, 혹은 컴파일 시간이 없는 것(보편적으로)을 말하지만, 최근 경험을 통한 나의 느낌으론, 가장 큰 장점은 **[Duck typing](http://en.wikipedia.org/wiki/Duck_typing)**을 자연스럽게[^natural_ducktyping] 작성할 수 있다는 것이다.  

![이미지 출처: http://coronalabs.com/]({{ site.url }}/attachments/duck_typing.png)  
<sub>이미지 출처: http://coronalabs.com/</sub>

Dynamic Type은 단지 아무 객체나 메서드에 전달 할 수 있다는 것을 뜻하는 것은 아니다. **객체가 누구인지보다 무엇을 할 수 있는지를 생각하게 한다.**  

비유적으로 설명하자면,  
Static type의 세상에선 누군가 요리를 하기 위해서 자격증을 발급받아야 하지만,
Dynamic Type의 세상에선 요리를 배우기만 하면된다. 즉, 요리를 시키기 위해서 요리사 자격증을 확인해야하는지, 요리를 할 수 있는지를 알아야하는지의 차이이다.  

자격증 확인 없이 어떻게 요리를 시킬 수 있는가? 합의(Contract)다. 행동을 하기위해 무리한 계층(Class Hierarchy)을 만들라고 강요하지 않는다. 계층을 통한 추상화가 아닌 합의를 통한 추상화를 허용한다. 이것이 구현(혹은 수정)과 확장에 더 높은 유연함을 준다. 게다가 높은 생산성과 쉬운 유지보수가 지향점인 OOP의 기본적인 원칙들을 해치는 것도 아니다.  

보통 Static Type을 지지하는 사람들은 요리사 자격증을 확인하는 것에 안전함을 느낄지 모른다. Type이 없다면 '팜므파탈이 남자를 요리하다'의 의미를 '남자를 재료로 음식을 만들다'로 오해할 수 있다는 지적이다. 하지만 이런 문제의 대부분은 인터페이스 설계가 잘못된것이다. [참조 - Contracts in Python](http://www.artima.com/intv/pycontract.html)  

타입오류는 프로그램에서 발생할 수 있는 수많은 오류 중 하나이다. 그 타입오류를 걱정하여 Static Type을 선호한다면 당연히 올바른 테스트를 작성하는 사람(조직)일테고, 올바른 테스트를 작성한다면 Dynamic Type의 타입오류는 테스트를 통해 검출될 수 있다. Duck Typing의 추상화가 이해하기 어려운 코드를 만든다고 생각한다면 좋은 문서를 제공하면 된다. 그리고 좋은 테스트는 좋은 문서가 된다.  

> Duck Typing은 Duck Typing이 없었다면 발견하지 못했을 추상화를 볼 수 있게 해준다. 이 추상화에 의존할 때 애플리케이션의 위험성은 줄어들고 유연성은 증가한다. 유지보수 비용이 줄어들고 쉽게 수정할 수 있게 된다. *참고*[^refered_book]

물론 언제나 Dynamic Type이 항상 옳은 것은 아니다.(라고 말해야할 것만 같다.) 하지만 일반적인 프로그램 디자인에 대한 기회비용은 상당히 높다. 무엇보다 코드를 쓸 때 자유로움을 느끼게해준다.  

------

[^undefined_method]: 사실 지금 생각해보면 대부분 `NoMethodError: undefined method 'xxx' for nil:NilClass`로 Java의 `java.lang.NullPointerException`과 유사한 실수의 에러다.  
[^natural_ducktyping]: 물론 Static Type에서도 Duck Typing을 할 수 있지만, **굳이** Template이나 Generic 구조를 사용해야한다.  
[^refered_book]: Practical Object-Oriented Design in Ruby, Sandi Metz  
