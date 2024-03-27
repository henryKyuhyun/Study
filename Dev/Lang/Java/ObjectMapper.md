# ObjectMapper?

## ObjectMapper 란?

### ->JSON 형식을 사용할 때, 응답들을 직렬화 하고 요청들을 역직렬화 할 때 사용하는 기술입니다.

### → ObjectMapper 는 생성 비용이 비싸기 때문에 bean / static 으로 처리하는 것을 권장한다.

참고 : 

### JSON(JavaScript Object Notation)

⇒ **key : value 쌍으로 이루어진 데이터 객체를 전달하기 위해 사람이 읽을 수 있는 텍스트를 사용하는 포멧이다.** 본래 자바스크립트 언어로부터 파생되어 자바스크립트 구문 형식을 따르지만 플랫폼과 언어 독립적 데이터 형식이다. 따라서 프로그래밍언어나 플랫폼에 독립적이므로 C, Java 과 Pyhton 등에서 JSON 데이터 생성을 위하 코드를 각자 가고지고 있다. JS를 제외한 언어는 라이브러리를 사용해야하는 경우가 많다.

### 직렬화 (Serialize)

⇒ 데이터를 전송하거나 저장할 때 바이트 문자열이야 하기 때문에 객체들을 문자열로 바꿔주는것. 
**Object → String 문자열**

### 역직렬화(Deserialize)

⇒ 데이터가 모두 전송된 이후, 수신측에서 다시 문자열을 기존의 객체로 회복시켜 주는 것
**String 문자열 → Object**

스프링부트의 경우 spring-boot-starter-web 에 기본적으로 Jackson 라이브러리가 있어서 Object ↔ JSON 간 변환은 자동으로 처리된다(Jackson 라이브러리란 자바에서 고수준의 JSON처리기이다)

@RestController 의 경우, 요청과 응답이 내부적으로 직렬화/역직렬화가 되는데 이는 Jackson 라이브러리가 있기 때문이다