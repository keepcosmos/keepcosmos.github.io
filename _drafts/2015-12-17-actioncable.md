---

layout: post
title: Realtime Web을 위한 ActionCable 소개
category: "dev"
description: ""
modified: 2015-12-19
comments: true
tags: [actioncable, realtime web, rails]

---

이 글은 루비 [크리스마스 달력](http://ruby-korea.github.io/advent-calendar/)에 참여하는 글입니다.

<iframe width="560" height="315" src="https://www.youtube.com/embed/KJVTM7mE1Cc" frameborder="0" allowfullscreen></iframe>
<p></p>
RailsConf 2015에서 Rails 5.0에 추가될 기능을 크게 3가지 발표했습니다. API, Turbolink, ActionCable 이 세가지가 그것인데요, ActionCable은 [가을에 나온다고 하였지만 아직도 많은 준비가 필요해보이는](https://github.com/rails/rails/pull/22585) Rails 5.0의 추가될 websocket 컴포넌트입니다.

기존에는 Rails만으로 Realtime Web(혹은 비슷한 것)을 구현하기 위해선 Polling과 Long Polling 그리고 SSE(ActionController::Live)를 사용했습니다. ActionCable을 살펴보긴 전에 먼저 각각 장단점을 살펴보면 다음과 같습니다.

polling의 경우 짧은 시간간격으로 서버에 요청을 보내는 방식입니다. 네트워크 상황에 따라 1, 2번의 요청이 실패한다고 해도 큰 문제가 없기 때문에 Long Polling에 비해서 안정적이라고 생각할 수도 있습니다. 무엇보다 구현하기 쉽다는 것이 가장 큰 장점입니다. 실제 Basecamp에서는 약 10년간 polling을 사용하였다고 하네요. 하지만 수많은 클라이언트들이 2, 3초의 간격으로 요청을 보낸다면 서버에 큰 부담이 될것입니다.

Long polling의 경우에는 polling과 비슷하지만 특정한 주기로 서버에 요청을 보내는 것이 아닌 request timeout을 사용하는 것입니다. 요청이 오면 서버 timeout이 일어날 때까지 해당 요청을 잡아둡니다. timeout이 일어나기 전에 변경사항이 생긴다면 응답을 보내주는 방식이죠. polling보다는 영속적으로 연결된다는 느낌이 듭니다. polling에 비해 request를 처리하는 부담이 적고 오래된 브라우져에서도 잘 작동하기 때문에 Facebook에서는 모두 Long polling을 사용하고 있다고 합니다. 하지만 polling에 비해 구현이 복잡하다는 단점이 있습니다.

Rails에서는 이 두가지 방법 말고도 [ActionController::Live](http://api.rubyonrails.org/classes/ActionController/Live/SSE.html)(Rails 4.0)을 사용한 SSE(Server Sent Event)를 구현할 수도 있습니다. 서버가 클라이언트에게 단방향으로 메시지를 보낼 수 있는 기능입니다. HTTP 프로토콜위에서 동작하니 기존 Rails Controller를 사용해 쉽게 구현될 수 있지만 IE는 지원하지 않기도하고, 서버 일방향 커뮤니케이션이라는 제한도 있습니다. 

물론, [Faye](http://faye.jcoglan.com/), [EventMachine](https://github.com/eventmachine/eventmachine)등 멋진 Ruby 라이브러리들도 존재합니다. 
[Pusher](https://pusher.com/)라는 SaaS형 서비스를 사용할 수도 있습니다.

기존에도 좋은 옵션들이 있었지만 이번 Rails 5.0에서는 Realtime web을 위한 ActionCable이 소개되었습니다. Rails 프레임워크에 ActionCable이 추가되서 얻을 수 있는 장점은 무엇일까요?

> If you can make WebSockets even less work than polling,<br/>
> why wouldn't you do it?<br/>
> -DHH

ActionCable을 통해 폴링을 구현하는 것보다 더 쉽게 웹소켓을 사용할 수 있도록 만들겠다는 것입니다.

ActionCable은 RealTime Web을 위한 Full-Stack 컴포넌트로 클라이언트 사이드 프레임워크(coffeescript)와 서버 사이드 프레임워크(ruby)를 모두 제공해줍니다. 서버사이드는 웹소켓 핸들링을 위한 [faye-websocket](https://github.com/faye/faye-websocket-ruby)과 멀티 쓰레드를 관리하기 위한[celluloid](https://github.com/celluloid/celluloid)를 사용하여 만들어져있습니다.

무엇보다 큰 장점이라고 한다면, Rails MVC의 한 부분이 아님에도 기존 컴포넌트들을 특별한 제약없이 사용할 수 있다는 것입니다. ActionCable 서버에서도 ActiveRecord를 통해 기존의 모델들을 읽고 쓸 수 있고, 뷰를 렌더링 할 수도 있습니다.

위에서 언급했던 것처럼, ActionCable은 기존 Rails MVC모델로 HTTP request처리를 않고 WebSocket 프로토콜을 핸들링합니다. 때문에 별도의 서버(ActionCable server)를 사용합니다. Rails 서버가 queue(redis)에 메시지를 넣고 ActionCable server가 이것을 확인하는 방식입니다.

![ActionCable architecture]({{ site.url }}/attachments/actioncable.png)

<sup>출처: https://www.zweitag.de/en/blog/technology/lets-build-a-chat-with-actioncable</sup>

메시지가 전달되는 시나리오는 이렇습니다.

1. 클라이언트는 ActionCable server에 커넥션(websocket)을 맺습니다. 커넥션이 맺혀진 클라이언트는 Consumer라 부릅니다. 서버는 `ActionCable::Connection::Base` 모듈을 상속받아 인증을 구현할 수 있습니다.

2. Cunsumer는 메시지를 전달받을 수 있는 Channel에 연결합니다. 이 때 Cunsumer는 subscriber로서의 역할을 하게됩니다. Channel은 목적에 따라 `ActionCable::Channel::base`를 상속받아 통해 서버 사이드를 구현합니다.

3. 다른 클라이언트가 Rails server로 요청(HTTP)을 보내게 되면 Rails server는 ActionCable server로 메시지를 전달합니다. Rails server가 publisher가 됩니다.  
    1. 메시지를 전달하는 과정에서 Queue를 사용하게 되는데 Redis의 pub/sub 기능을 사용합니다. Rails server 안에서 `ActionCable.server.broadcast` 메서드를 통해 메시지를 Redis로 publish 합니다.
    2. ActionCable서버는 Redis로부터 메시지를 Subscribe를 받고 Consumer에게 전달하게됩니다.

상세 구현된 예제는 [rails/actioncable-example](https://github.com/rails/actioncable-examples)에서 확인 할 수 있습니다.

현재(2015년 12월 19일) ActionCable은 rails/rails에 merge되기 전 [Guide 작업](https://github.com/rails/rails/issues/22673)과 railtie를 수정하는 작업을 진행하고 있습니다. 내년 초 즈음 ActionCable이 포함된 Rails 5.0을 기대해 볼 수 있을겁니다.
