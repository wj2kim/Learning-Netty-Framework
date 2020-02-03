# Learning-Netty-Framework

채팅 어플리케이션 개발을 위해 공부한 Netty Framework의 관한 내용 정리

# 네티소개

## 네티란

- 자바 네트워크 어플리케이션 프레임워크이다.
- 이벤트 기반 ( Event-Driven) 이며 비동기 방식을 지원한다.
- OIO ( Old I/O), New I/O 모두 지원한다.

## 네티의 주요 특징

### 동기와 비동기

- 여기서 살펴보고자 하는 동기와 비동기는 함수 또는 서비스의 호출 방식에 관한 내용이다.
    - 동기식 호출
    로그인시 호출자가 인증 요청을 하고 컨트롤러/서비스/데이터베이스에서 인증 처리 로직이 완료 된 후 인증 결과를 돌려주는 로직의 서비스가 있을때 호출자는 호출 시작 이후 부터 종료 까지는 호출의 결과를 확인 할 수 없다. 즉 호출이 종료 되고 난 이후 처리 결과를 확인 할 수 있다. 
    이처럼 서비스 처리가 완료된 이후 처리 결과를 알 수 있는 방식을 동기식 호출 이라고 한다. 
    특정 메소드나 서비스가 이와 같은 호출 방식을 지원한다면 그것을 동기식 호출을 지원한다고 표현한다.
    - 비동기식 호출
    비동기식  호출에서는 호출자의 인증 요청을 서비스가 수신하면 서비스는 스레드의 인증 요청을 위한 함수를 다른 스레드에 등록한다. 실제 인증 처리가 완료 되지 않아도 서비스 호출 종료가 이루어 지고 그 응답으로 티켓을 호출자에게 전달한다. 
    호출자의 관점에서는 인증 요청에 대한 결과를 수신하지는 못했지만 인증 요청 자체는 완료되었기 때문에 인증 요청 결과를 기다리는 시간에 다른 작업을 수행할 수 있고 필요한 시기에 서비스로부터 받은 티켓을 사용하여 요청한 인증 처리가 완료되었는지 확인 할 수 있다.

### 블로킹과 논블로킹

- 소켓의 동작 방식은 블로킹 모드와 논블로킹 모드로 나뉜다. 블로킹은 요청한 작업이 성공하거나 에러가 발생하기 전까지는 응답을 돌려주지 않는 것을 말하며 논블로킹은 요청한 작업의 성공 여부와 상관없이 바로 결과를 돌려주는 것을 말한다. 이때 요청의 응답값에 의해서 에러나 성공 여부를 판단한다.
- 블로킹 소켓은 ServerSocket, Socket클래스. 논블로킹 소켓은 ServerSocketChannel, SocketChannel 클래스를 사용한다.
    - 블로킹 소켓 
    클라이언트가 서버로 연결 요청을 보내면 서버는 연결 수락을 하고 클라이언트와 연결된 소켓을 새로 생성하는데 이때 해당 메소드의 처리가 완료되기 전까지 스레드의 블로킹이 발생하게 된다. 클라이언트가 연결된 소켓을 통해서 서버로 데이터를 전송하면 서버는 클라이언트가 전송한 데이터를 읽기 위하여 read 메소드를 호출하고 이 메소드의 처리가 완료되기 전까지 스레드가 블로킹 된다. 클라이언트 또 한 마찬가지로 서버에서 클라이언트로 전송한 데이터를 읽기 위한 메소드에서 블로킹이 발생한다.
    * 데이터 입출력에서 스레드의 블로킹이 발생하기 때문에 동시에 여러 클라이언트에 대한 처리가 불가능
    이러한 문제점을 해결하는 방법은 연결된 클라이언트별로 각각 스레드를 할당하는 방법이 있다. 하지만 동시에 접속 요청을 하는 상황에서 대기시간이 길어진다는 점과 서버의 스레드 수가 증가해서 자바의 힙 메모리 부족으로 인한 out of memory 오류가 발생 할 수 있다. 
    이러한 오류를 피하기 위해 스레드 풀을 사용하기도 한다. 클라이언트가 서버에 접속하면 서버 소켓으로 부터 클라이언트 소켓을 얻어온다. 다음으로 스레드 풀에서 가용 스레드를 하나 가져오고 해당 스레드에 클라이언트 소켓을 할당한다. 이와 같은 구조에서는 동시에 접속 가능한 사용자 수가 스레드 풀에 지정된 스레드 수에 의존하는 현상이 발생한다. 스레드 풀의 크기를 자바 힙이 허용하는 최대 한도에 도달할 때까지 늘리는 것이 합당한지 두가지 관점에서 고민해 봐야한다. 
    1. 힙에 할당된 메모리가 크면 클수록 가비지 컬렉션이 수행되는 횟수는 줄어들지만 수행시간은 상대적으로 길어진다. 
    2. 컨텍스트 스위칭 시 수많은 스레드가 CPU 자원을 획듣하기 위하여 경쟁하면서 CPU자원을 소모하고 실제로 사용할 CPU 자원이 적어지게 된다.
    - 논블로킹 소켓
    앞써 말한 블로킹 모드의 소켓은 read, write, accept 메소드 등과 같은 입출력 메소드가 호출되면 처리가 완료될 떄까지 스레드가 몸추게 되어 다른 처리를 할 수 없었다. 이러한 단점을 해결하는 방식이 논블로킹 소켓이다. 논블로킹 소켓은 구조적으로 소켓으로 부터 읽은 데이터를 바로 소켓에 쓸 수가 없다. 이를 위해서 각 이벤트가 공유하는 데이터 객체를 생성하고 그 객체를 통해서 각 소켓 채널로 데이터를 전송한다.
    블로킹 소켓과 다르게 소켓 채널을 먼저 생성하고 사용할 포트를 바인딩한다. 
    여러 클라이언트는 하나의 스레드에 등록된다. 그리고 각가은 Selector 라는 객체에 채널을 등록하고 Selector는 자신에게 등록된 채널에 변경 사항이 발생했는지 검사를 하고 이에 응답한다.

### 동기와 비동기 / 블로킹과 논블로킹의 대한 간략 정리

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/76a958a3-4a3f-41b9-92c2-a85850dc3950/Screen_Shot_2020-02-02_at_5.19.08_PM.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/76a958a3-4a3f-41b9-92c2-a85850dc3950/Screen_Shot_2020-02-02_at_5.19.08_PM.png)

<img src="https://s3-us-west-2.amazonaws.com/secure.notion-static.com/76a958a3-4a3f-41b9-92c2-a85850dc3950/Screen_Shot_2020-02-02_at_5.19.08_PM.png" width="90%"></img>

### 이벤트 기반 프로그래밍

- 전통적으로 사용자 인터페이스가 포함된 프로그램에 많이 사용된다. ( 예를 들어 마우스 클릭에 반응하는 코드) 이와 같이 각 이벤트를 먼저 정의해두고 발생한 이벤트에 따라서 코드가 실행되도록 프로그램을 작성하는것이 이벤트 기반 프로그래밍이다. * 앞서 말한 논블로킹 소켓의 Selector를 사용한 I/O이벤트 감지 역시 이벤트 기반 프로그램의 한 종류이다.
- 추상화 수준 : 이벤트를 나눌 떄 작은 단위로 나누었는지 큰 단위로 나누었는지의 정도를 말하는 것.
- 네트워크 프로그램에서 이벤트가 발생하는 주체는 소켓이다
- 발생하는 이벤트는 크게 소켓 연결 / 데이터 송수신으로 나눌 수 있다.
- 서버는 클라이언트 연결을 수락하기 위해 서버 소켓을 생성하고 포트를 서버에 바인딩 한다.
- 이로서 클라이언트의 연결을 수락하고 데이터 송수신 할 소켓을 생성할 수 있고 이때부터 송수신이 가능해진다.
- 일반적으로 네트워크 프로그램에서 소켓은 IP주소와 포트를 가지고 있고 양방향 네트워크 통신이 가능한 객체이다.
- 만약 소켓이나 연결된 스트림에 문제가 발생한다면 새로운 소켓을 생성하고 데이터를 전송하는 예외 처리 코드가 작동되게 해야한다.
- 네티는 데이터의 읽기 쓰기를 위한 이벤트 핸들러를 지원한다. (ChannelInboundHandlerAdapter 등 )
- 네티는 데이터를 소켓으로 전송하기 위해 채널에 직접 기록하는 것이 아니라 데이터 핸들러를 통해서 기록한다. 이 와같은 방법은 서버 애플리케이션의 코드를 클라이언트 애플리케이션에서 재사용 할 수 있는 장점이 있다.
- 데이터 핸들러는 에러 이벤트도 같이 정의하기 때문에 에러처리에 용이하다.

# 네티 구성요소

## Bootstrap

- 부트스트랩은 네티로 작성한 네트워크 애플리케이션이 시작할때 가장 처음으로 수행되는 부분이다. 부트스트랩 설정은 크게 이벤트 루프, 채널의 전송 모드, 채널 파이프라인으로 나뉜다.
    - 이벤트 루프는 소켓 채널에서 발생한 이벤트를 처리하는 스레드 모델에 대한 구현이 담겨있다.
    - 부트스트랩에서 설정한 소켓의 모드에 따라 사용되는 이벤트 루프의 구현체가 달라지기도 한다. 사용할 소켓 채널의 모드에 따라서 블로킹, 논블로킹, epoll로 지정가능하다. epoll은 가장 빠르지만 linux에서만 동작한다.
    - 채널 파이프라인은 연결된 채널에서 사용할 데이터 핸들러에 대한 내용을 포함한다 ( 코덱도 포함)

### Bootstrap 정의

- 네트워크 애플리케이션의 동작 방식과 환경을 설정을 도와주는 클래스이다.
- 예를 들어 네트워크에서 수신한 데이터를 단일 스레드로 데이터베이스에 저장하는 네트워크 애플리케이션을 구현하고자 한다면 이 애플리케이션은 8088번 포트에서 데이터를 수신하고 소켓모드는 NIO를 사용한다. 이와 같은 요구사항을 부트스트랩으로 설정 가능하다. NIO소켓 모드를 지원하는 서버 소켓 채널, 데이터를 변환하여 데이터베이스에 저장하는 데이터 처리 이벤트 핸들러, 애플리케이션이 사용할 8088번 포트 바인딩, 단일 스레드를 지원하는 이벤트 루프에 대한 설정이 필요하다.

### Bootstrap 구조

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/76a958a3-4a3f-41b9-92c2-a85850dc3950/Screen_Shot_2020-02-02_at_5.19.08_PM.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/76a958a3-4a3f-41b9-92c2-a85850dc3950/Screen_Shot_2020-02-02_at_5.19.08_PM.png)

- 위의 표는 부트스트랩이 지원하는 설정 목록이다.
- 통상적으로 네트워크 애플리케이션은 서비스를 제공할 네트워크 포트, 네트워크 전송에 사용할 소켓 모드와 소켓 옵션, 소켓의 데이터를 처리하는 스레드, 그리고 애필르케이션에서 사용하는 프로토콜로 구성된다.
- 서버 애플리케이션과 클라이언트 애플리케이션의 구분은 소켓 연결을 요청하느냐 아니면 대기하느냐에 따른 구분이다.
- ServerBootstrap 클래스는 클라이언트 접속을 대기할 포트를 설정하는 메소드가 추가되있다.
- 네티의 부트스트랩 설정을 통해서 데이터 처리 코드를 변경하지 않고 BlockingServer 또는 NonBlockingServer와 동일한 동작을 하는 애플리케이션을 작성할 수 있다. 사용자는 단순히 사용할 입출력 모드에 해당하는 소켓 채널 클래스를 설정하기만 하면 변경이 완료 된다.

### ServerBootstrap 과 Bootstrap

- 부트스트랩 클래스가 빌더패턴 ( 인수없는 생성자 생성 후 객체 추가) 으로 작성되어 있으므로 인수 없는 생성자로 객체를 생성하고 group, channel 과 같은 메소드로 객체를 초기화 한다. ( 이와 같은 객체 초기화 방법은 생성자를 사용한 객체 생성 방법에 비해 코드가 더 간결하고 설정하려는 내용이 명확해진다.
- ServerBootstrap 객체에 서버 애플리케이션이 사용할 두 스레드 그룹을 설정하는데 첫 번째 스레드 그룹은 클라이언트의 연결을 수락하는 부모 스레드 그룹이고 두번째 스레드 그룹은 연결된 클라이언트의 소켓으로 부터 데이터 입출력 및 이벤트 처리를 담당하는 자식 스레드 그룹니다. 그룹은 NioEventLoopGroup 클래스의 객체로 설정했다. 클래스 생성자의 인수 사용되는 숫자는 스레드  그룹 내에서 생성할 최대 스레드 수를 의미한다. 1 = 단일 스레드로 동작.
- 인수 없는 생성자는 사용 할 스레드 수를 서버 애플리케이션이 동작하는 하드웨어 코어 수를 기준으로 결정한다
- 서버 애플리케이션의 소켓 모드를 블로킹 모드로 변경하여면 OioEventLoopGroup으로 변경하고 소켓 채널의 클래스를 OioServerSocketChannel로 지정하면 된다. ( 매우 간단함 )
- 이와같이 부트스트랩은 추상화 모델을 제공하기 때문에 네트워크 애플리케이션 개발자가 확장성과 유연성을 동시에 만족하는 코드를 작성할 수 있다.
- ServerBootstrap API
    - 부트스트랩의 주요 기능인 이벤트 루프, 채널의 전송 모드, 채널 파이프라인 등은 ServerBootstrap의 API로 매칭된다.
    - ServerBootstrap → .group 메소드 인수 두개 가능
    - Bootstrap → .group 메소드 인수 한개 가능
    - Channel 은 소켓의 입출력 모드를 설정하는 메소드이다. 이 Channel 메소드는 abstractBootstrap 추상 클래스의 구현체인 ServerBootstrap 과 Bootstrap 클래스에 모두 존재하는 API고 channel 메소드에 등록된 소켓 채널 생성 클래스가 소켓 채널을 생성한다.
    - channelFactory
        - channel 매소드와 동일하게 입출력 모드를 설정하는 API이다. 네티가 제공하는 ChannelFactory 인터페이스의 구현체로는 NioUdtProvider가 있다.
    - handler
        - 서버 소켓 채널의 이벤트를 처리할 핸들러 설정 API이다. ServerBootstrap의 handler 메소드로 등록된 이벤트 핸들러는 서버 소켓 채널에서 발생한 이벤트만을 처리한다. ( 비슷하지만 다른 메소드로는 childHandler 가 있음 )
    - childHandler
        - 클라이언트 소켓 채널로 송수신 되는 데이터를 가공하는 데이터 핸들러 설정 API 이다. handler 메소드와 childHandler메소드는 ChannelHandler 인터페이스를 구현한 클래스를 인수로 입력 한다. 이를 통해 등록되는 이벤트 핸들러는 서버에 연결된 클라이언트 소켓 채널에서 발생하는 이벤트를 수신하여 처리한다.
    - option
        - 서버 소켓 채널의 소켓 옵션을 설정하는 API 이다. 소켓 옵션이란 소켓의 동작 방식을 지정하는 것을 말한다. ( 커널( 송신 버퍼, 수신 버퍼) 에서 사용되는 값을 변경)
        - 네티는 자바 가상머신을 기반으로 동작하기 때문에 자바에서 설정할 수 있는 소켓 옵션을 모두 설정 할 수 있다. 예: SO_KEEPALIVE (운영체제에서 지정된 시간에 한번씩  keepalive 패킷을 상대방에게 전송한다. SO_LINGER (소켓을 닫을 때 커널의 송신 버퍼에 전송되지 않은 데이터의 전송 대기시간을 지정한다) 등등이 있다.
    - childOption
        - 앞서 말한 옵션은 소켓 채널에 소켓 옵션을 설정하지만 childOption 메소드는 서버에 접속한 클라이언트 소켓 채널에 대한 옵션을 설정한다. 예 : SO_LINGER ( close 메소드를 호출한 이후 제어권은 애플리케이션에서 운영체제로 넘어가게 된다. 이때 커널 버퍼에 아직 전송되지 않은 데이터가 남아 있으면 이 옵션의 사용 여부와 타임 아웃 값의 설정에 따라 처리할 수 있다. 서버 애플리케이션이 동작 중인 운영체제에서 TIME_WAIT이 많이 생기는 것을 방지하기 위해 사용하거나 Proxy 서버가 동작중인 운영체제에서 TIME_WAIT가 발생하는것을 방지하기 위해 사용함.
- Bootstrap API
    - option 과 childOption 같이 두 가지 설정을 제공하지 않는다. 즉 option 메소드만 존재한다 (child가 없기 때문에)
    - group
        - 위와 같이 그룹 메소드는 소켓 채널의 이벤트 처리를 위한 이벤트 루프를 설정한다. Bootstrap의 group 메소드는 단 하나의 이벤트 루프만 설정 가능하다. ( 클라이언트는 서버에 연결할 소켓채널 하나만 가지고 있기 때문)
    - channel
        - channel 메소드는 소캣 채널의 입출력 모드를 설정한다. 클라이언트 소켓만 서정 가능하다. 예: NioSocketChannel.class
    - channelFactory
        - ServerBootstrap 의 channelFactory 와 동일한 동작을 수행한다.
    - handler
        - 클라이언트  소켓에서 발생하는 이벤트를 수신하여 처리한다. 보통 ChannelInitializer 클래스의 handler 메소드를 통해 등록된다.

## Channel Pipeline

- 채널 파이프라인은 채널에서 발생한 이벤트가 이동하는 통로다. 이 통로를 통해 이동하는 이벤트를 처리하는 클래스를 이벤트 핸들러하고 하고 이벤트 핸들러를 상속받아서 구현한 구현체들을 코덱이락고 한다.

## Event Handler

## Codec

## Event Loop

## ByteBuf

# 네티 응용

### How to Review

- Make two passes over the PR if it's substantial.
    - On the first pass, come to an understanding of the code change at a high level.
    - On the second pass, pay more attention to semantic details.

# 참고 개념

### OIO/NIO

- 자바에서는  JDK 1.3까지 블로킹방식의 I/O 만을 지원했다. 이후 JDK 1.4 부터는 NIO 라는 논블로킹 I/O API가 추가 되었다. 입출력과 관련된 기능을 제공하고 소켓도 입출력 채널의 하나로써 NIO API를 사용할 수 있으며 NIO API  를 통해서 블로킹과 논블로킹 모드의 소켓을 사용할 수 있다.

### SCTP

- SCTP는 TCP의 연결지향 및 전송 보장 특성과 UDP의 메시지 지향 특성을 모두 갖추고 있다.  TCP는 세 방향 핸드셰이크를 통해서 연결을 수립하지만 SCTP는 네 방향 핸드셰이크(INT-ACK, COOKIE-ECHO) 를 통해서 연결을 수립하고, 이때 연결 정보에 쿠키를 삽입하여 DOS와 같은 네트워크 공격으로 부터 보호한다. ㅐㅔㅐㅔ123123123

### flip() 메소드

### Executor

### So_LINGER

# Examples

    var commentCount = 0;

You might suggest that this be a `let` instead of `var`.
