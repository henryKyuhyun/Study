# JPA개념 정리

### JPA (Java Persistence API)

- Java Persistence API 의 약자로 Java 언어를 통해 DB와 같은 영속 계층을 처리하고자 하는 스펙이다.
- JPA를 이해하기 위해 **ORM(Object Relational Mapping) 이라는 기술을 선행**해야 한다.
- **[ORM](https://www.notion.so/ORM-6020d8371d6445ee85ad431718e7ce6a?pvs=21) 을 Java언어에 맞게 사용하는 ‘스펙’이다**. 따라서 ORM이 조금 더 상위 스펙이되고, JPA는 java언어에 국한된 개념이라고 볼 수 있다.
- JPA 는 단순한 스펙이기 때문에 해당 스펙을 구현하는 구현체마다 회사의 이름이나 프레임워크의 이름이 다르게 된다. 다양한 프레임워크가 있지만 가장 유명한 것은 ‘hibernate’이다.
- 기존 EJB에서 제공되던 Entity Bean을 대체하는 기술이다.
- **[ORM](https://www.notion.so/ORM-6020d8371d6445ee85ad431718e7ce6a?pvs=21) 이기 때문에 자바 클래스**와 DB Table을 매킹한다.(SQL 맵핑하진 않는다!)

![Screenshot 2024-02-26 at 11.39.31 am.png](JPA%E1%84%80%E1%85%A2%E1%84%82%E1%85%A7%E1%86%B7%20%E1%84%8C%E1%85%A5%E1%86%BC%E1%84%85%E1%85%B5%203f1eb473c22e4ad3923a908acf0dc8b8/Screenshot_2024-02-26_at_11.39.31_am.png)


## ORM (Object Relational Mapping)

- ORM 은 객체지향과 관련이 있다. ‘**객체지향 패러다임을 관계형 데이터베이스에 보존하는 기술**’ 이라고 할 수 있다.
- 패러다임 입장에서 보면 ‘객체지향 패러다임을 관계형 패러다임으로 매핑(Mapping)해주는 개념’
- ORM의 시작은 ‘객체지향’ 의 구조가 ‘관계형 데이터베이스’와 유사하다는 점에서 시작한다. 객체지향 언어 중 ‘클래스’ 를 사용하는 언어는 특히 그러한 경우인데 예를 들어 ‘ 클래스’ 장치를 사용하는 OOP 프로그래밍 언어들은 어떠한 데이터의 구조를 잡기 위해 우선적으로 클래스를 설계한다.
- 관계형 데이터베이스를 다루는 입장에서는 클래스는 아니나’ 테이블’ 을 설계한다. 새로운 테이블에는 컬럼을 정의하고 컬럼에 맞는 데이터 타입을 지정해서 데이터를 보관하는 틀을 만든다는 의미에서 클래스와 상당히 유사하다.
- **ORM은 자바 객체를 DB테이블로 매핑함으로써 객체관 관계를 바탕으로 SQL을 자동으로 생성하지만, Mapper는 SQL을 명시해주어야 한다.**

- 클래스와 테이블이 유사하듯 '인스턴스'와 'Row(레코드 또는 튜플)'도 상당히 유사하다. **객체지향에서는 클래스에서 인스턴스를 생성해 인스턴스라는 '공간'에 데이터를 보관**하는데, **테이블에서는 하나의 'Row'에 데이터를 저장**하게 된다. 여기서의 유일한 차이는 '객체'라는 단어가 '데이터 + 행위(메서드)'라는 의미라면 **'Row'는 '데이터'만**을 의미한다는 점이 다를 뿐이다. (데이터베이스에서는 '개체(Entity)'라는 용어를 사용한다.)
- 이와 같이 객체지향과 관계형 데이터베이스(RDBMS)는 유사한 특징을 갖는다. 이런 특징에 기초해 **'객체지향을 자동으로 관계형 데이터베이스에 맞게' 처리해주는 기법에 대해 아이디어를 내기 시작했고 그것이 ORM의 시작이다.**

# # Spring Data JPA 그리고 JPA

1. Spring Boot는 JPA의 구현체 중 'Hibernate'라는 구현체를 이용한다. 이는 Open Source로 ORM을 지원하는 프레임워크이다.
2. **"Spring Data JPA"는 Hibernate를 스프링 부트에서 쉽게 사용할 수 있는 추가적인 API들을 제공한다.**
3. 다른 프레임워크도 그러하듯 Hibernate도 독립된 프레임워크이다. 따라서 스프링 부트가 아닌 스프링에서도 Hibernate와 연동하여 사용이 가능하다.)
4. JPA는 Application과 JDBC사이에서 동작한다.
5. 개발자가 JPA를 사용하면, JPA 내부에서 JDBC API를 사용해 SQL을 호출하여 DB와 통신한다.



### JPA 장점

- 특정 데이터베이스에 종속성 감소
    - 데이터베이스 종속성 감소: 각 데이터베이스는 고유의 쿼리문을 가지고 있어, 데이터베이스의 변경이 어렵습니다. 그러나 JPA는 데이터 접근 계층을 추상화하여 제공하므로, 설정 파일에서 사용하는 데이터베이스만 지정하면 됩니다.
- 객체 지향적 프로그래밍 및 생산성 향상
    - JPA는 개발자가 SQL을 직접 작성하는 데이터베이스 설계 중심의 접근 방식에서 벗어나, Java 객체에 집중할 수 있도록 지원합니다. 이를 통해 객체 지향적인 프로그래밍이 가능하며, 생산성을 향상시킬 수 있습니다. 예를 들어, 테이블의 컬럼이 수정되면, 해당 컬럼에 매핑된 Java 클래스만 변경하면 됩니다. 이렇게 함으로써 소스 코드와 데이터베이스 사이의 결합도를 낮추고 유지보수성을 향상시킬 수 있습니다.

### JPA 단점

- 복잡한 쿼리 처리의 어려움
    - JPA는 복잡한 쿼리를 처리하는데 제한적이 면이 있습니다. 특히, 복잡한 조인이나 서브 쿼리등을 사용하는 경우, JPA 의 기능만으로는 한계가 있을 수 있습니다.
- 성능 저하의 위험성
    - JPA는 개발자가 직접 작성한 쿼리가 아닌, 자동으로 생성되는 쿼리를 사용하므로, 경우에 따라서는 예상치 못한 성능 저하가 발생할 수 있습니다. 따라서 JPA를 사용하는 상황에서는 쿼리의 성능을 주기적으로 확인하고, 필요한 경우 직접 쿼리를 최적화하는 작업이 필요합니다.
- 학습 곡선
    - JPA는 방대한 기술을 포함하고 있으며, 이를 완전히 이해하고 사용하기 위해서는 상당한 시간과 노력이 필요합니다.

### JPA 동작원리
![Screenshot 2024-02-23 at 1 22 51 am](https://github.com/henryKyuhyun/Study/assets/118201123/95146210-6597-45ed-980e-42e8aac4d74f)



1. Entity Manage 생성 : JPA를 사용하기 위해 가장 먼저 엔티티 매니저를 생성합니다. 이 엔티티 매니저는 어플이케이션과 테이터베이스 사이에서 객체와 관계형 데이터를 관리합니다.
2. 트랜잭션 관리 : 데이터베이스의 작업은 트랜잭션 단위로 이루어집니다. JPA는 이러한 트랜잭션을 관리해주는 역할을 합니다.
3. SQL 실행 : JPA는 어플리케이션에서 필요한 SQL을 자동으로 생성하고 실행합니다. 이를 통해 개발자는 SQL작성에 대한 부담을 줄일 수 있으며, 객체 지향적인 코드를 작성할 수 있습니다.
4. 1차 캐시와 동일성(Identify)보장 : JPA 는 1차 캐시를 제공합니다. 동일한 트랜잭션 안에서는 같은 엔티티를 반환함으로써 데이터의 일관성을 유지합니다.
5. Lazy Loading 과 즉시로딩(Eager Loading) : JPA 는 연관된 엔티티를 언제 로딩할지 선택할 수 있습니다. 이를 통해 어플리케이션의 성능을 향상시킬 수 있습니다.
6. JPQL : JPA는 객체 지향 쿼리 언어인 JPQL을 제공합니다. 이를 통해 개발자는 SQL이 아닌 객체 지향적인 방식으로 데이터를 다룰 수 있게됩니다. 

이처럼 JPA는 개발자가 객체 지향적인 프로그래밍을 할 수 있게 도와주며, 반복적인 코드 작성을 줄여주고, 데이터베이스 설계  변경 시에도 유연하게 대체 가능합니다.

### Entity

⇒ Entity 는 데이터베이스의 테이블을 대응하는 클래스로, 예를 들어 데이터베이스에 ‘item’이라는 테이블이 있다면 이를 대응하는 ‘Item.java’ 클래스가 존재할 것입니다. 이러한 클래스에는 @Entity 어노테이션이 붙어있으며 이 어노테이션을 통해 JPA가 해당 클래스를 관리하게 됩니다. 

### Entity Manager Factory

⇒ 엔티티 매니저 인스턴스를 생성하고 관리하는 주체입니다. 어플이케이션 실행 시점에 한 번만 생성되며, 사용자로부터 요청이 들어올 떄마다 엔티티 매니저를 생성하게 됩니다.

### **Entity Manager**

⇒ **엔티티 매니저는 Persistence Context에 접근하여 데이터베이스 작업을 수행하는 객체입니다. 이는 내부적으로 DB Connection을 이용하여 데이터베이스에 접근하게 됩니다.**

### **영속성 컨텍스트(Persistence Context)**

⇒ **영속성 컨텍스트는 엔티티를 영구적으로 저장하는 환경을 제공합니다. 이는 엔티티 매니저를 통해 접근이 가능하며, 이를 통해 엔티티의 생명주기를 관리하게 됩니다.**

## Entity Life Cycle

![Screenshot 2024-02-23 at 1 23 07 am](https://github.com/henryKyuhyun/Study/assets/118201123/d4049168-bac9-436b-a6dd-8cacfc589e84)


**비영속(Transient)** : JPA와 전혀 관계가 없는 상태입니다. 엔티티 객체를 새로 생성했지만 아직 영속성 컨텍스트나 데이터베이스와는 연관되지 않은 상태를 말합니다.

**영속(Persistent)** : 영속성 컨텍스트에 저장된 상태입니다. **엔티티 매니저**를 통해 엔티티를 영속성 컨텍스트에 저장하면, 해당 엔티티는 '영속' 상태가 됩니다. 이 상태의 엔티티는 데이터베이스와 동기화되며, 엔티티 매니저가 관리하는 범위에 속하게 됩니다.

**준영속(Detached)** : **영속성 컨텍스트에 저장되었다가 분리된 상태**입니다. 엔티티가 영속성 컨텍스트에서 분리되면, 더 이상 JPA의 관리를 받지 않게 됩니다. **따라서 변경 감지 및 자동 저장 등의 기능을 사용할 수 없게** 됩니다.

**삭제(removed)** :삭제된 상태입니다. 영속성 컨텍스트에 저장된 엔티티를 삭제하면, 해당 엔티티는 '삭제' 상태가 됩니다. 이 상태의 엔티티는 데이터베이스에서 삭제되며, 엔티티 매니저의 관리 범위에서 제외됩니다.

위의 4가지 상태는 엔티티 매니저의 메소드를 통해 상태를 변경할 수 있습니다. 예를 들어, 'persist()' 메소드를 통해 엔티티를 영속 상태로 만들 수 있고, 'detach()' 메소드를 통해 엔티티를 준영속 상태로 만들 수 있습니다. 또한 'remove()' 메소드를 통해 엔티티를 삭제 상태로 만들 수 있습니다.

### Persistence Context 사용시 이점

- **1차 캐시**

**영속성 컨텍스트는 내부적으로 1차 캐시를 가지고 있어서, 이를 통해 DB 접근 횟수를 줄이고 성능을 향상시킬 수 있습니다.**

- **동일성 보장**

**영속성 컨텍스트가 동일성을 보장하기 때문에, 같은 트랜잭션 내에서는 같은 엔티티에 대한 동일성이 보장됩니다. 이를 통해 불필요한 DB 접근을 줄일 수 있습니다.**

- **트랜잭션을 지원하는 쓰기 지연**

**영속성 컨텍스트는 트랜잭션을 지원하는 쓰기 지연 기능을 제공합니다. 이는 DB에 즉시 반영하지 않고, 트랜잭션을 커밋하는 시점에 한꺼번에 SQL을 DB에 보내는 방식입니다. 이를 통해 네트워크 비용이나 DB CPU 사용량 등을 줄일 수 있습니다.**

- **변경 감지**

**영속성 컨텍스트는 엔티티의 변경을 감지(더티 체킹)하는 기능을 제공합니다. 이를 통해 개발자가 직접 DB에 변경사항을 반영하는 코드를 작성하지 않아도 됩니다.**

**이 외에도 영속성 컨텍스트는 트랜잭션 롤백이나 지연 로딩(lazy loading) 등 다양한 기능을 제공합니다. 이러한 기능들을 이해하고 활용하면 JPA를 통한 개발이 더욱 효율적이게 활용할 수 있다.**

![Screenshot 2024-02-23 at 1 23 19 am](https://github.com/henryKyuhyun/Study/assets/118201123/375841d9-63de-435a-9562-0ff90f6578a1)


# JPA- save()  , SaveAndFlush()

JPA의 save()와 saveAndFlush() 메소드는 데이터를 데이터베이스에 저장하는 역할을 하지만, 그 동작 방식에는 차이가 있습니다.

save(): 이 메소드를 호출하면, 해당 엔티티를 영속성 컨텍스트에 저장하고, 그 ID를 반환합니다. 이 때, 실제로 데이터베이스에 저장되는 것은 아닙니다. 트랜잭션이 종료되는 시점에 영속성 컨텍스트의 변경 내용이 데이터베이스에 반영되는데, 이를 'flush'라고 합니다. 따라서, save() 메소드를 호출한 직후에는 바로 데이터베이스에 변경이 반영되지 않습니다.
saveAndFlush(): 이 메소드는 save()와 마찬가지로 엔티티를 영속성 컨텍스트에 저장하지만, 추가적으로 바로 'flush'를 수행하여 변경 내용을 즉시 데이터베이스에 반영합니다. 따라서, saveAndFlush() 메소드를 호출하면 트랜잭션이 종료되지 않아도 바로 데이터베이스에 변경이 반영됩니다.
어떤 메소드를 사용할지는 상황에 따라 달라집니다.

save(): 대부분의 경우에 save() 메소드를 사용하면 충분합니다. 여러 번의 데이터베이스 작업을 한 번의 트랜잭션으로 묶어서 처리할 수 있으므로 성능적으로 이점이 있습니다.
saveAndFlush(): 특정 작업을 즉시 데이터베이스에 반영해야 할 때 사용합니다. 예를 들어, 엔티티 저장 후 바로 다른 쿼리를 실행해야 하거나, 트랜잭션을 종료하지 않고 바로 결과를 확인해야 하는 경우에 유용합니다.

save()와 saveAndFlush() 모두 엔티티를 저장하거나 수정하는 데 사용될 수 있습니다. 즉, 게시판 글을 등록하거나 수정하는 데는 두 메서드 모두 사용할 수 있습니다. 선택은 주로 트랜잭션의 범위나, 데이터베이스에 반영되는 시점에 따라 결정됩니다.

save(): 이 메서드는 주로 새로운 게시글을 등록할 때 사용할 수 있습니다. save()를 호출하면, 해당 엔티티는 영속성 컨텍스트에 저장되고, 트랜잭션이 종료되는 시점에 데이터베이스에 반영됩니다. 이렇게 하면 여러 개의 작업을 한 번의 트랜잭션으로 묶을 수 있으므로 성능적으로 이점이 있습니다.
saveAndFlush(): 이 메서드는 특정 작업을 즉시 데이터베이스에 반영하고 싶을 때 사용합니다. 수정된 게시글을 바로 반영하고, 그 결과를 확인하고 싶은 경우에 saveAndFlush()를 사용할 수 있습니다. 이 메서드를 호출하면, 해당 엔티티가 영속성 컨텍스트에 저장되고 바로 'flush'가 일어나서 데이터베이스에 반영됩니다.
따라서, 게시판 글을 등록하거나 수정하는 데 어떤 메서드를 사용할지는 주로 작업의 성격과 트랜잭션의 범위에 따라 결정되며, 두 작업 모두 save() 또는 saveAndFlush()를 사용할 수 있습니다.
