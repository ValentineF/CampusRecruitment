# Lambda系列-排序

## Lambda的Comparator和排序

### 0.自定义一个基础类

```java
@Data
public class Person {

    private String name;

    private Integer age;
}
```

### 1.非lambda方式的排序(内部类)

```java
Collections.sort(personList, new Comparator<Person>() {
            @Override
            public int compare(Person o1, Person o2) {
                return o1.getAge().compareTo(o2.getAge());
            }
        });
```

### 2.Lambda基本排序

```java
Collections.sort(personList, (o1, o2) -> o1.getAge().compareTo(o2.getAge()));
```

### 3.Comparator的comparing

```java
Collections.sort(personList, Comparator.comparing(Person::getAge));
```

### 4.反转排序

Comparator的reversed()方法

```java
Collections.sort(personList,Comparator.comparing(Person::getName).reversed());
```

### 4.多条件组合排序

普通if逻辑

```java
Collections.sort(personList,(o1,o2)->{
            if(o1.getName().equals(o2.getName())){
                return o1.getAge().compareTo(o2.getAge());
            }else{
                return o1.getName().compareTo(o2.getName());
            }
        });
```

Compartor的thenComparing

```java
Collections.sort(personList,Comparator.comparing(Person::getName).thenComparing(Person::getAge));
```

## 参考资料

![Java8：Lambda表达式增强版Comparator和排序](https://wizardforcel.gitbooks.io/java8-tutorials/content/Java%208%20Lambda%20%E8%A1%A8%E8%BE%BE%E5%BC%8F%E5%A2%9E%E5%BC%BA%E7%89%88%20Comparator%20%E5%92%8C%E6%8E%92%E5%BA%8F.html)