# WebSocket + Redis 적용기

### 유저 Redis Caching 처리 후 채팅방의 대화내용도 caching 처리해서 찾아오는 비용을 줄여봣습니다.

# 결과 ⇒

<img src="/img/WebSocket_Redis1.png">

### 채팅방 대화내역 확인

<img src="/img/WebSocket_Redis2.png">


### 그리고 Redis-Cli + Terminal 로 결과확인

<img src="/img/WebSocket_Redis3.png">


### 첫번째 에러와의 만남 ⇒

첫번째는 직렬화문제 두번째는 이런에러가발생한다 JSON 데이터를 역직렬화 하는 과정에서 만예쌍치 못한 에러가 발생했고 오류메시지를 보면 Unexpected token (START_OBJECT), expected START_ARRAY: need JSON Array to contain As.WRAPPER_ARRAY type information for class java.lang.Object

**"Unexpected token (START_OBJECT), expected START_ARRAY"라고 말하고 있어요. 이는 JSON 파서가 객체(`{}`)를 만났지만, 배열(`[]`)을 기대했다는 의미입니다**

2024-03-18 20:12:07.579 ERROR 96045 --- [nio-8081-exec-8] c.y.y.exception.GlobalControllerAdvice   : Error occurs org.springframework.data.redis.serializer.SerializationException: Could not read JSON: Unexpected token (START_OBJECT), expected START_ARRAY: need JSON Array to contain As.WRAPPER_ARRAY type information for class java.lang.Object
at [Source: (byte[])"{"id":20,"chatRoomEntity":{"id":21,"chatRoomName":"jijiji1","users":[{"id":16,"userName":"admin322","password":"$2a$10$MGnzUZGIMhAIwgkP8vQKUufmztXPeJe/8NzPC5XexHlah0DDMoiNi","nation":"SouthKorea","birthOfDayAndTime":"1999-07-06T09:20:00","bloodType":"B형","gender":"FEMALE","registeredAt":"2024-02-16T07:45:12.873+00:00","updatedAt":null,"deletedAt":null,"chatRooms":[{"id":12,"chatRoomName":"내 채팅방5","users":[16],"messageEntities":[],"registeredAt":"2024-03-05T01:43:29.691+00:00"},{"id":18"[truncated 2236 bytes]; line: 1, column: 1]; nested exception is com.fasterxml.jackson.databind.exc.MismatchedInputException: Unexpected token (START_OBJECT), expected START_ARRAY: need JSON Array to contain As.WRAPPER_ARRAY type information for class java.lang.Object

그래서 인터넷을 찾아보는중 ObjectMapper 를 수정해보라는 글을보았고 

```java
    public void saveMessage(Integer chatRoomId, Message message){
        String key = getKey(chatRoomId);
        log.info("SET Messages to Redis {} : {}", key, message);
        try{
            String messageJson = objectMapper.writeValueAsString(message);
            messageRedisTemplate.opsForList().rightPush(key, messageJson);
            messageRedisTemplate.expire(key,MESSAGE_CACHE_TTL);
        } catch (JsonProcessingException e) {
            throw new RuntimeException(e);
        }
    }

    public List<Message> getMessages(Integer chatRoomId) {
        String key = getKey(chatRoomId);
        List<String> messagesList = messageRedisTemplate.opsForList().range(key, 0, -1);
        if (messagesList != null && !messagesList.isEmpty()) {
            return messagesList.stream().map(obj -> {
                try {
                    return objectMapper.readValue(obj.toString(), Message.class);
                } catch (JsonProcessingException e) {
                    throw new RuntimeException(e);
                }
            }).collect(Collectors.toList());
        }
        return new ArrayList<>();
    }
```

SaveMessage 의 objectMapper.writeValueAsString(message); 

를 호출하여 Message객체를 JSON 문자열로 변환합니다. 이렇게 변환된 문자열이 Redis 에 저장되며, 이 과정은 Java 객체를 Redis와 같은 데이터 스토리지 시스템에 저장하기 위해 필요한 과정입니다. JSON 포맷으로 변환함으로써 , 객체의 상태를 텍스트 기반 포멧으로 저장할 수 있게됩니다.

getMessages 메소드에서 Redis에서 읽어온 JSON 문자열을 다시 Message 객체로 변환합니다.

objectMapper.readValue(obj.toString(), Message.class); 

호출을 통해 이 작업이 이루어지며 이 과정은 저장된 데이터를 다시 Java 객체로 변환하여 어플리케이션에 사용할 수 있게만듭니다.

**이러한 직렬화 및 역직렬화 과정을 통해, 애플리케이션은 객체의 상태를 손쉽게 저장하고 검색할 수 있게 됩니다. 또한, JSON 포맷을 사용함으로써 데이터 포맷이 표준화되어 다른 시스템과의 통신에서도 용이하게 데이터를 교환할 수 있습니다.**

```java
    @Bean
    public RedisTemplate<String, Message> MessageRedisTemplate(RedisConnectionFactory connectionFactory) {
        RedisTemplate<String, Message> redisTemplate = new RedisTemplate<>();
        redisTemplate.setConnectionFactory(connectionFactory);

        redisTemplate.setKeySerializer(new StringRedisSerializer());

        ObjectMapper objectMapper = new ObjectMapper();
        objectMapper.registerModule(new JavaTimeModule());
        objectMapper.disable(SerializationFeature.WRITE_DATES_AS_TIMESTAMPS);

        objectMapper.activateDefaultTyping(objectMapper.getPolymorphicTypeValidator(), ObjectMapper.DefaultTyping.NON_FINAL);

        GenericJackson2JsonRedisSerializer serializer = new GenericJackson2JsonRedisSerializer(objectMapper);
        redisTemplate.setValueSerializer(serializer);
        return redisTemplate;
    }
```

1.  objectMapper.registerModule(new JavaTimeModule());  를 이용해 Java8 날짜 및 시간 API를 올바르게 처리합니다. 
    1. Java 8에 도입된 새로운 날짜 및 시간 API(ex: LocalDateTime, ZonedDateTime)는 기본적으로 JSON 직렬화 및 역직렬화를 지원하지 않습니다(그래서 User에 Redis를 할때 dateOfBirth에 에러발생한 경험이있다..)따라서 JavaTimeModule 을 ObjectMapper 에 등록함으로써 이러한 타입들이 올바르게 처리되게 합니다.
2. objectMapper.disable(SerializationFeature.WRITE_DATES_AS_TIMESTAMPS); 를 비활성화해서 날짜와 시간을 타임스탬프가 아닌, 사람이 읽을 수 있도록 처리했습니다. 
    1. 이 설정은 날짜와 시간을 숫자 타임스템프가 아니라 사람이 읽기 쉬운 문자열 형태(ex:”2011-09-21T11:13:29Z”)로 직렬화하기 위한 것입니다. 이는 데이터의 가독성을 높이고 클라이언트 측에서 날짜 및 시간 처리를 용이하게 합니다.
3.  objectMapper.activateDefaultTyping(objectMapper.getPolymorphicTypeValidator(), ObjectMapper.DefaultTyping.NON_FINAL);
 **설정은 객체 타입 정보를 포함하도록 하여 역직렬화 시 타입 안정성을 높힙니다.**
    1. JSON으로 직렬화된 객체를 역직렬화할 때, 객체의 실제 클래스 타입을 알아야만 정확한 타입으로 복원할 수 있습니다. 특히 다형성을 지원하는 객체(예: 상속 구조를 가진 객체)의 경우 이 정보가 없으면 역직렬화 과정에서 문제가 발생할 수 있습니다. activateDefaultTyping을 사용하면, 직렬화된 JSON에 클래스 타입 정보를 포함시켜 이러한 문제를 해결할 수 있습니다
    

이에러뿐아니라 직렬화역직렬화, 데이터타입, Null값이 보이는등 정말 하루정도의 시간을 소요하였으며 코드가 엉망진창이되서 새롭게 지우고 다시 구상을하며 Redis의 가장기본적인 에러라할수있는 직렬화,역직렬화 를 경험해보았습니다.