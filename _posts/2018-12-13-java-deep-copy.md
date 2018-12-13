---
layout: post
title:  "객체가 들어있는 컬렉션(ArrayList or HashSet 등) 목록 깊은 복사(Deep copy) 하기."
date:   2018-12-13 18:36:00 +0900
categories: Java
comments: true
tags: [Java, Deep Copy]
---

---

# 객체가 들어있는 컬렉션(ArrayList or HashSet 등) 목록 깊은 복사(Deep copy) 하기.

Java의 Collection은 기본적으로 얕은 복사(shallow copy)는 제공하나 깊은 복사기능은 제공하지 않는다.
즉, 원본 목록과 복제된 목록에 저장된 객체가 동일하고 그렇다는것은 Java 힙 공간에서 동일한 메모리 위치를 가리켜 문제가 될 수 있다.

예를 들면 직원 정보가 들어있는 ArrayList의 생성자를 사용하여 복제시엔 직원 객체 정보는 그대로 있으므로 복제된 list의 객체를 수정시에 같이 영향을 받게 된다. 

아래 예제이다.


```java
import java.util.ArrayList;
import java.util.Iterator;
import java.util.List;

public class CollectionCloningTest {
    public static void main(String args[]) { 
        List<Employee> org = new ArrayList<>();
        org.add(new Employee("MA", "Manager"));
        org.add(new Employee("AH", "Developer"));
        org.add(new Employee("JM", "Developer"));

        // 생성자를 통해 copy list 생성.
        List<Employee> copy = new ArrayList<>(org);

        System.out.println("ORG LIST : " + org);
        System.out.println("COPY LIST : " + copy);

        Iterator<Employee> itr = org.iterator();
        while (itr.hasNext()) {
            itr.next().setDesignation("Specialist");
        }

        //org list 에 해당하는 값을 바꿨으나 copy list 에 들어있는 객체값도 같이 바뀌게 된다.
        System.out.println("ORG LIST : " + org);
        System.out.println("COPY LIST : " + copy);
    }
}

class Employee {
    private String name;
    private String designation;

    public Employee(String name, String designation) {
        this.name = name;
        this.designation = designation;
    }

    public String getDesignation() {
        return designation;
    }

    public void setDesignation(String designation) {
        this.designation = designation;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    @Override
    public String toString() {
        return String.format("%s: %s", name, designation);
    }
}
```

- 출력값 : org list 에 해당하는 값을 바꿨으나 copy list 에 들어있는 객체값도 같이 바뀌게 된다.
> ORG LIST : [MA: Manager, AH: Developer, JM: Developer]<br>
COPY LIST : [MA: Manager, AH: Developer, JM: Developer]<br>
ORG LIST : [MA: Specialist, AH: Specialist, JM: Specialist]<br>
COPY LIST : [MA: Specialist, AH: Specialist, JM: Specialist]

복제본이 단순 복사이고 힙의 동일한 Employee 개체를 가리 키기 때문에 원본 컬렉션의 Employee 개체 ( "Specialist"로 변경된 지정)가 복제 된 컬렉션에도 반영되어 있음을 확인할 수 있다. 
<br>이 문제를 해결하려면 Collection을 반복할 시 Employee 객체를 깊게 복제해야하며 그 전에 Employee 객체의 복제 메소드를 재정의해서 넣어야한다.

### 해결방법
1) 객체 복사를 할 수 있도록 하는 인터페이스 구현 (Employee가 Cloneable 인터페이스를 구현하도록 한다.)
2) Employee 클래스에 clone() 메소드 추가


/**
	1. 기존 Employee 클래스에 Cloneable을 구현하도록 추가.
	2. clone 메소드 구현.
*/
```java
class Employee implements Cloneable {
    private String name;
    private String designation;

    public Employee(String name, String designation) {
        this.name = name;
        this.designation = designation;
    }

    public String getDesignation() {
        return designation;
    }

    public void setDesignation(String designation) {
        this.designation = designation;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    @Override
    public String toString() {
        return String.format("%s: %s", name, designation);
    }

    @Override
    public Employee clone() {
        Employee clone = null;
        try {
            clone = (Employee) super.clone();
        } catch (CloneNotSupportedException e) {
            throw new RuntimeException(e);
        }
        return clone;
    }
}
```

생성자를 사용하여 List를 복사하는 대신 다음 코드를 사용하여 Java에서 콜렉션 전체 복사

```java
public class CollectionCloningTest {
    public static void main(String args[]) { // deep cloning Collection in Java
		....위와 같은 로직...
			
		//deep copy 로직
		List<Employee> deepCopy = new ArrayList<>(org.size());
		Iterator<Employee> iterator = org.iterator();
		while (iterator.hasNext()) {
			deepCopy.add(iterator.next().clone());
		}

		//deepCopy list 에 해당하는 값을 바꿨으나 org list 에 있는 객체값이 영향을 받지 않는다..
		deepCopy.get(0).setDesignation("Expert");

		System.out.println("ORG LIST : " + org);
		System.out.println("DEEP COPY LIST : " + deepCopy);
	}
}

```

참고자료
 - https://javarevisited.blogspot.com/2014/03/how-to-clone-collection-in-java-deep-copy-vs-shallow.html#axzz5ZXZJA1NR


[jekyll-docs]: https://jekyllrb.com/docs/home
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-talk]: https://talk.jekyllrb.com/

---

