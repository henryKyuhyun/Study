# DeepCopy And ShallowCopy

## 깊은 복사(Deep copy) : ‘실제 값(Value)’ 을 새로운 메모리 공간에 복사하는 것을 의미한다

## 얕은 복사(Shallow copy) : ‘참조 값(주소 값)’ 을 복사한다는 것을 의미한다.

얕은 복상의 경우 주소 값을 복사하기에 참조하고 있는 실제 값은 같다

### **깊은 복사(Deep Copy)**

- 정의 : 객체의 실제 내용(값)을 새로운 메모리 공간에 복사하여, 완전히 독립된 복사본을 만드는 것입니다.
- 특징 :
    - 객체를 복사할 때, 해당 객체와 인스턴스 변수까지 모두 복사하는 방식
    - 전부를 복사하여 새 주소에 담기 때문에 **참조를 공유**하지 않는다.
    - 복사된 객체는 원복 객체와 완전히 **다른 주소를 가리키게 됩니다**
    - 원본 객체의 값이 변경되어도 **복사된 객체에는 영향을 주지 않습니다**.
- 적용 예 : ArrayList 의 clone() 메서드를 오버라이드해 깊은 복사를 구현할 수 있습니다.

```java
class Person implements Cloneable {
    private String name;
    private int age;
    private Address address; // 참조 타입의 필드

    public Person(String name, int age, Address address){
        this.name = name;
        this.age = age;
        this.address = address;
    }

    // 깊은 복사를 위한 메서드
    @Override
    protected Object clone() throws CloneNotSupportedException {
        Person cloned = (Person) super.clone();
        cloned.address = (Address) address.clone(); // Address도 깊은 복사 수행
        return cloned;
    }
    
    // 나머지 getters와 setters
}

class Address implements Cloneable {
    private String street;
    private String city;

    // constructor, getters, setters
    
    // 깊은 복사를 위한 메서드
    @Override
    protected Object clone() throws CloneNotSupportedException {
        return super.clone();
    }
}

```

사용 예 : 

```java
public class DeepCopyExample {
    public static void main(String[] args) {
        Address address = new Address("123 Street", "City");
        Person originalPerson = new Person("John", 30, address);

        // 깊은 복사를 수행합니다.
        Person clonedPerson = (Person) originalPerson.clone();

        // originalPerson의 Address와 clonedPerson의 Address는 
				//메모리상 다른 위치에 저장됩니다.
    }
}

```

여기서 Person 클래스와 Address 클래스 모두 Cloneable 인터페이스를 구현하고 clone() 메서드를 재정의하여 깊은 복사가 가능하도록 만들었습니다. cloned.address = (Address) address.clone(); 부분에서 Address 객체도 깊은 복사를 하는 것을 볼 수 있습니다.

깊은 복사는 주로 원본 객체와 복사본이 서로 독립적인 상태를 유지해야 할 때 사용합니다. 예를 들어, 원본 객체의 상태 변경이 복사본에 영향을 미치지 않아야하는 경우나, 복사본을 수정해도 원본이 변경되지 않아야 할 때 유용합니다.

참고로 위 코드 예시에서 clone() 메서드는 CloneNotSupportedException을 던질 수 있으므로, 주의해야합니다

### **얕은 복사(Shallow Copy)**

- 정의 : 객체의 참조 값(주소값)을 복사하여, 원본 객체와 같은 메모리 주소를 참조하는 복사본을 만드는 것입니다.
- 특징 :
    - 복사된 객체와 원복 객체는 같은 메모리 주소를 가리키게 되어 **서로 영향을 줍니다**
    - 원본 객체의 값이 변경되면 **복사된 객체의 값도 같이 변경**됩니다.
- 적용 예 : 단순히 객체의 참조 변수에 객체를 할당하는 방식(Object newObj = oldObj;) 으로 얕은 복사가 일어납니다

```java
class Person {
    private String name;
    private int age;

    public Person(String name, int age){
        this.name = name;
        this.age = age;
    }

    // getters and setters
    public String getName() {
        return name;
    }
    
    public void setName(String name) {
        this.name = name;
    }
    
    public int getAge() {
        return age;
    }
    
    public void setAge(int age) {
        this.age = age;
    }
}

public class ShallowCopyExample {
    public static void main(String[] args) {
        Person originalPerson = new Person("John", 30);

        // 얕은 복사를 수행합니다.
        Person copiedPerson = originalPerson;

        System.out.println("Original Person Name: " + originalPerson.getName()); // "John"
        System.out.println("Copied Person Name: " + copiedPerson.getName()); // "John"

        // 복사된 객체의 데이터를 변경합니다.
        copiedPerson.setName("Doe");

        // 바뀐 이름을 출력하여 확인합니다.
        System.out.println("Original Person Name after changes: " + originalPerson.getName()); // "Doe"
        System.out.println("Copied Person Name after changes: " + copiedPerson.getName()); // "Doe"
    }
}

```

**위 코드에서 첫 두 줄의 출력 결과는 `"John"`이지만, `copiedPerson.setName("Doe");` 실행 후의 출력 결과는 두 변수 모두 `"Doe"`를 출력합니다. 이는 두 객체가 같은 메모리 주소를 참조하고 있기 때문에 발생하는 현상입니다.**

**얕은 복사는 특히 객체 내부에 다른 객체를 참조하는 참조형 필드가 있을 때 주의해야 합니다. 그 이유는 참조형 필드도 얕은 복사가 되어 원본과 복제본 모두 같은 객체를 참조하기 때문입니다. 따라서 한 객체에서 참조하는 객체의 데이터를 변경하면, 다른 객체에서 참조하는 객체의 데이터도 함께 바뀌게 됩니다.**

**얕은 복사는 객체의 생성 비용이 크지 않고, 공유해도 문제가 없는 단순한 경우에 사용하며, 빠르고 간단하다는 장점이 있습니다. 하지만, 참조하는 객체의 독립성을 보장할 수 없다는 단점도 있습니다.**