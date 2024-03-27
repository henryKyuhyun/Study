# SSE (Server-Sent_Events)

나는 개인적으로 구현하고싶은것중 하나가 내가 로그인을 하고있을때 어떤유저가(같은 사이트에 회원가입이되어있는) 현재 로그인의 여부를 알수있는 로직을 구현하고싶었다 그래서 알아보았다

방법은 아래 몇가지가있다.

1. 전통적인 Client-server 모델의 HTTP 통신
2. Short Polling

1. Long Polling
2. Server Sent Event(SSE)

하지만 내가아는 가장 기본적 Client-Server HTTP 통신은 이런 기능을 구현하기가 어렵다고 생각한다 왜냐하면 클라이언트의 요청이있어야만 서버가 응답을하기떄문..그래서 다른 3가지를 중점적으로 알아보기로 한다.

### **Short Polling**

이건 클라이언트가 **서버에게 지속적으로 HTTP요청을 보내어 업데이트 여부**를 확인하는거다.

이방법은 간단하고 구현이 쉽지**만 불필요한 요청이 발생하고 서버 부하가 높아질수 있는 단점이있다.**
<img src="/img/sseimg1.png">
<img src="/img/sseimg2.png">

처음엔 false로 초기화해둠 다음에 업데이트가되서 true가되면 리턴

<img src="/img/sseimg3.png">

옆 코드를 보면 while 루프를 이용해 지속적으로 get요청을 보내고 요청에 응답을 받아서 데이터업데이트 여부확인,

클라이언트 코드에서 URL객체 생성 후 [localhost:8080/data](http://localhost:8080/data) 주소 연결, HttpURLConnection 클래스 사용해 HTTP GET 요청보내고 , 응답받아오는 코드

응답내용을 **BufferReader, InputStreamReader 이용하여 읽고**   **StringBuffer로 문자열**추가

**`if (content.toString().equals("Data updated"))`**
 조건문을 사용하여 서버에서 응답으로 전송한 데이터가 업데이트되었는지 여부를 판단,

**`Thread.sleep(1000);`**
 코드는 1초간 대기하는 것으로, 서버에 반복적으로 요청을 보내는 주기를 정해줌

// 모르는거 :

**`org.apache.logging.log4j.message.Message`**는 Log4j 2에서 사용되는 메시지 인터페이스입니다.

Log4j는 로그를 기록하기 위한 강력한 프레임워크로, **로그 레벨에 따라 로그 메시지를 기록하고 다양한 출력 방식으로 전송할 수 있습니다.** 

Log4j 2에서는 다양한 로그 메시지 인터페이스를 제공하며, **`org.apache.logging.log4j.message.Message`**는 그 중 하나입니다.

**`Message`** 인터페이스는 로그 메시지의 내용을 나타내는 메서드(**`getFormattedMessage()`**, **`getParameters()`**, **`getThrowable()`**)와 로그 레벨(**`getLevel()`**) 등을 정의합니다. **`Message`** 인터페이스를 구현한 클래스들은 Log4j에서 로그 메시지를 생성할 때 사용됩니다.

### **Long Polling**

클라이언트와 서버 간의 실시간 통신방식인데 HTTP 요청에 대한 서버응답을 유지하고, 서버측에서 업데이트 될떄까지 연결 유지

장점 :

- 실시간 데이터 전송가능. (채팅, 실시간 게임등에 유리)
- 서버 부하 감소(데이터가 준비 될때까지 응답을 보내지않아서)

단점:

- 불필요한 연결 유지( 많은 수의 클라이언트가 연결 유지하면 서버 자원낭비)
- 지연 시간 : 데이터가 업데이트 될때마다 연결 다시 설정해야해서 지연시간 증가 / 따라서 실시간성이 중요한경우는 WebSocket과 같은 기술이 좋음
- 복잡함 구현어렵고 유지보수어려움

<img src="/img/sseimg4.png">

<img src="/img/sseimg5.png">

위 간단한 코드는 GET 요청이 들어오면 ‘MessageService’에서 가져온 메시지가 있는지 확인 후 없다면 5초대기, 있다면 가져오고 없다면 다시 GET요청

### **Server-Sent Events(SSE)**

서버와 한번 연결을 맺고나면 일정 시간동안 서버에서 변경이 발생할 때마다 데이터를 전송받는 방법입니다.

특히 **웹어플리케이션에서 서버로부터 이벤트를 수신하여 처리하는데 유용**

장점

- 클라이언트와 서버 간의 양방향 통신을 위해 WebSocket보다 단순한 프로토콜을 사용합니다.
- SSE는 **일반 HTTP 요청**을 사용하기 때문에, **프록시 서버와 방화벽에서 블록**되지 않습니다.
- SSE는 **WebSocket**보다 일반적인 브라우저에서 지원됩니다.

단점

- SSE는 일반적으로 **한 방향의 통신만 지원**합니다. 클라이언트가 서버에 데이터를 전송하려면 **추가적인 API를 사용해야합니다.**
- SSE는 데이터 전송을 위해 **텍스트 형식을 사용하며, 대용량 데이터를 전송하는 데 적합하지 않습니다.**
- SSE는 HTTP 연결을 유지해야하므로 **서버에 부하를 유발할 수 있습니다**.

SSE는 실시간으로 데이터를 전송하는데 유용한 기술입니다. 클라이언트와 서버 간의 양방향 통신이 필요하지 않을 때는 SSE가 WebSocket보다 단순한 구현 방법이 될 수 있습니다. 하지만 대용량 데이터를 처리해야하는 경우나 양방향 통신이 필요한 경우 WebSocket을 고려해야합니다.

<img src="/img/sseimg6.png">

# **SSE 통신 개요**

SSE를 이용하는 통신의 개요는 다음과 같습니다.

<img src="/img/sseimg7.png">

### SSE는 단방향이기때문에 만약 실시간채팅을 원하면 webSocket 을 구현하는게 가장적합하다

```

@**Controller**
public class SSEController {
@**GetMapping**(value = "/sse", produces = **MediaType.TEXT_EVENT_STREAM_VALUE**)
public **Flux**<ServerSentEvent<String>> sse() {
    return **Flux**.interval(Duration.ofSeconds(1))
            .map(seq -> ServerSentEvent.<String>builder()
                    .id(String.valueOf(seq))
                    .event("message")
                    .data("SSE - " + LocalTime.now().toString())
                    .build());
}
}

// Flux 사용하기위한 의존성
implementation 'org.webjars.npm:flux:3.1.3'
**: 상태를 변경할 떄마다 이벤트를 생성하고 처리해주는거 (단방향)**
: 위코드에서 Flux.interval(Duration.ofSeconds(1) 은 1초마다 정수시퀸스를 생성하는 Flux생성하고 이 시퀸스는 map을 통해 ServerSentEvent 객체로 평가.

<!DOCTYPE html>
<html xmlns:th="[http://www.thymeleaf.org](http://www.thymeleaf.org/)">
<head>
<title>SSE Example</title>
</head>
<body>
	<h1>SSE Example</h1>
	<div id="sse">
		<p>Waiting for server-sent events...</p>
	</div>

<script type="text/javascript">
	**const** eventSource = new EventSource("/sse");

	eventSource.onmessage = function(event) {
		const data = event.data;
		const sseDiv = document.querySelector('#sse');
		sseDiv.innerHTML += '<p>' + data + '</p>';
		};
</script>
</body>
</html>
```

### Long Polling VS SSE

**Long Polling과 Server-Sent Events (SSE)는 *모두 비동기 방식*으로 서버와 클라이언트 간의 통신을 가능하게 하는 기술.**

하지만 **Long Polling**은 클라이언트가 서버에 요청을 보내고, 

서버는 즉시 응답을 반환하지 않고 일정 시간 동안 연결을 유지한 다음, 

새로운 데이터가 생성될 때까지 계속해서 연결을 유지하는 기술. 

이는 클라이언트에서 주기적으로 서버에 요청을 보내는 폴링 기술과 비슷하다. 서버는 새로운 데이터가 생성되면, 그때 응답을 반환하고, 클라이언트와의 연결을 끊는다.

**반면에, SSE는 서버**가 클라이언트에게 **지속적으로 데이터를 전송하는 방식으로 작동한다.** 

클라이언트는 서버와 연결을 유지하며, 서버는 새로운 데이터가 생성되면 해당 데이터를 클라이언트에게 즉시 전송. 

이는 클라이언트에서 서버로의 폴링을 방지하며, 서버 측에서 불필요한 응답을 보내지 않아도 된다.

따라서, Long Polling은 서버에서 데이터가 생성될 때까지 연결을 유지하는 반면, SSE는 지속적으로 데이터를 전송한다. 

SSE는 실시간 애플리케이션에서 많이 사용되며, Long Polling은 SSE보다 상대적으로 부담이 적은 애플리케이션에서 사용될 수 있다.

⇒ 

## 서버가 클라이언트에게 지속적 데이터를??? response를 지속적으로? 만약 변화(이벤트)가 없어서 보낼께없으면?

= > 이 경우에는 서버가 빈 이벤트를 주기적으로 보낸다 이를 ‘heartbeat or ping’ 이벤트라고 부른다. 변화가없어도 계속 저걸보내므로 서버와의 연결이 여전히 유효한지 확인 가능 , 이를 통해 서버와의 연결유지 가능 

Spring boot에서는 SseEmitter 클래스 이용해서 로직구현가능하다.

```

import java.io.IOException;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

**java.util.concurrent.Executors 클래스는 쓰레드 풀을 생성하는 데 사용됩니다. 위 코드에서는 Executors.newCachedThreadPool() 메소드를 사용하여 크기가 동적으로 조정되는 쓰레드 풀을 생성합니다.
ExecutorService는 쓰레드 풀에서 쓰레드를 관리하기 위한 인터페이스입니다. ExecutorService는 submit() 메소드를 이용하여 새로운 작업을 쓰레드 풀에 제출할 수 있습니다. 제출된 작업은 쓰레드 풀 내의 쓰레드에 의해 비동기적으로 실행됩니다.**

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.servlet.mvc.method.annotation.SseEmitter;

@**RestController**
public class StreamController {

private **ExecutorService** executor = Executors.new**CachedThreadPool**();

@GetMapping("/stream")
public SseEmitter stream() {
    **SseEmitter** emitter = new SseEmitter();
    executor.execute(() -> {

****        try {
            for (int i = 0; i < 10; i++) {
                emitter.send(SseEmitter.event().name("message").data("Data " + i));
                Thread.sleep(1000);
            }
            emitter.complete();
        } catch (IOException | InterruptedException e) {
            emitter.completeWithError(e);
        }
    });
    return emitter;
}
}

**위 코드에서는 executor.execute() 메소드를 사용하여 새로운 작업을 쓰레드 풀에 제출합니다. 
제출된 작업은 SseEmitter를 이용하여 SSE를 지속적으로 전송하는 작업입니다.**
**이렇게 생성된 쓰레드 풀은 스레드 수를 유동적으로 조정할 수 있기 때문에, 동시에 많은 클라이언트가 SSE를 수신할 경우에도 서버의 부하를 낮출 수 있습니다**
```

위 코드에서는 **`/stream`** 엔드포인트에 접속하면, **`SseEmitter`** 객체를 생성하여 반환합니다. 

**`SseEmitter`** 객체는 SSE를 지원하는 스프링의 클래스로, 서버에서 클라이언트로 데이터를 전송할 수 있는 메커니즘을 제공합니다.

그리고, **`executor`**를 사용하여 새로운 쓰레드에서 **`SseEmitter`**를 이용하여 SSE를 지속적으로 전송합니다. 

**`send()`** 메소드를 사용하여 데이터를 보내고, **`complete()`** 메소드를 호출하여 SSE 스트림을 종료합니다.

위 코드는 10개의 데이터를 1초 간격으로 전송하는 예제입니다. 클라이언트는 **`/stream`** 엔드포인트에 연결하면, 

1초 간격으로 "Data 0", "Data 1", ..., "Data 9"를 수신할 수 있습니다.

참고로, **`/stream`** 엔드포인트를 브라우저에서 열면, SSE를 지원하는 브라우저는 자동으로 SSE 스트림을 수신하여 데이터를 처리할 수 있습니다. 

예를 들어, Chrome 브라우저에서 F12를 누르고 Network 탭을 선택한 다음 **`/stream`** 엔드포인트를 호출하면, SSE 스트림을 확인할 수 있습니다.