---
title: Spring自动装配与cp命名空间DI优先级
date: 2020-05-30 14:13:40
categories:
- 技术博客
tags:
- Spring


---







***
##### 先说结论：
#### Spring使用自动装配时，同一个bean配置文件下，类型为引用的Spring自动装配中，其DI（dependency injection）优先级高于同一个标签中的c命名空间的DI优先级，而P命名空间的优先级高于自动装配。
#### 即
# DI（P-namespace） > DI（Autowiring） > DI（C-namespace）
***
*为减少代码篇幅长度，下文中的pojo类中使用了Lombok第三方jar包来简述代码*
*（Lombok这个神器应该很多小伙伴都知晓，这里作者就不阐述了，假如不知道其实也不影响）*


# 1、P命名空间的使用（Properties）

**Spring配置：**
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:p="http://www.springframework.org/schema/p"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd">

<!--    1、p注入-->
    <bean id="catttttttt" class="cn.BinGCU.pojo.AninalImpl.Cat" p:name="zhangsanCat" scope="singleton"/>
    <bean id="doggggg" class="cn.BinGCU.pojo.AninalImpl.Dog" p:name="zhangsanDog" scope="singleton"/>

    <bean id="personnnnnn" class="cn.BinGCU.pojo.Person" p:cat-ref="catttttttt" p:dog-ref="doggggg"/>


</beans>
```
**pojo：**
```java
import lombok.Data;

public interface Animal {
    void shout();
}

@Data
public class Cat implements Animal {
    private String name;

    @Override
    public void shout() {
        System.out.println("Cat: miao~");
    }
}


@Data
public class Dog implements Animal {
    private String name;

    @Override
    public void shout() {
        System.out.println("Dog: bark~");
    }
}


```



**Test：**
```java
        ApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");

        System.out.println(context.getBean("catttttttt", Cat.class));
        System.out.println(context.getBean("doggggg", Dog.class));
```

**Output：**
```output
Cat(name=zhangsanCat)
Dog(name=zhangsanDog)
```


# 2、C命名空间的使用（Constructor-args）

**Spring配置：**
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:p="http://www.springframework.org/schema/p"
       xmlns:c="http://www.springframework.org/schema/c"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd">

<!--    2、c注入-->
    <bean id="catttttttt" class="cn.BinGCU.pojo.AninalImpl.Cat" p:name="lisiCat"/>
    <bean id="doggggg" class="cn.BinGCU.pojo.AninalImpl.Dog" p:name="lisiDog"/>
	
    <bean id="personnnnn" class="cn.BinGCU.pojo.Person" c:cat-ref="catttttttt" c:dog-ref="doggggg"/>
</beans>
```

**pojo：**
```java
import lombok.Data;

public interface Animal {
    void shout();
}

@Data
public class Cat implements Animal {
    private String name;

    @Override
    public void shout() {
        System.out.println("Cat: miao~");
    }
}


@Data
public class Dog implements Animal {
    private String name;

    @Override
    public void shout() {
        System.out.println("Dog: bark~");
    }
}


@Data
public class Person {

    private Cat cat;
    private Dog dog;

    public Person(){}

    public Person(Cat cat, Dog dog) {
        this.cat = cat;
        this.dog = dog;
    }
}
```
**Test：**
```java
        ApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");

        Person personnnnn=  context.getBean("personnnnn", Person.class);
        System.out.println(personnnnn);
```
**Output**
```output
Person(cat=Cat(name=lisiCat), dog=Dog(name=lisiDog))
```
# 3、自动装配
**Spring配置：**
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        https://www.springframework.org/schema/context/spring-context.xsd">

	<!--    启动自动装配-->
    <context:annotation-config />

	<--这里的id（cat和dog）与Person类中的属性名一致，请看下文的pojo定义-->
	<bean id="cat" class="cn.BinGCU.pojo.AninalImpl.Cat"  p:name="wangwuCat"/>
    <bean id="dog" class="cn.BinGCU.pojo.AninalImpl.Dog"  p:name="wangwuDog"/>

    <bean id="person" class="cn.BinGCU.pojo.Person" />
</beans>

```

**pojo：**
```java
import lombok.Data;

public interface Animal {
    void shout();
}

@Data
public class Cat implements Animal {
    private String name;

    @Override
    public void shout() {
        System.out.println("Cat: miao~");
    }
}


@Data
public class Dog implements Animal {
    private String name;

    @Override
    public void shout() {
        System.out.println("Dog: bark~");
    }
}


@Data
public class Person {
    @Autowired
    private Cat cat;
    @Autowired
    private Dog dog;

    public Person(){}

    public Person(Cat cat, Dog dog) {
        this.cat = cat;
        this.dog = dog;
    }
    
    public void managePets(){
        System.out.println("Manage Dog and Cat");
        cat.shout();
        dog.shout();
    }
}
```
**Test：**
```java
        ApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");

        Person person = context.getBean("person", Person.class);
        person.managePets();
        System.out.println(person);
```
**Output**
```output
Manage Dog and Cat
Cat: miao~
Dog: bark~
Person(cat=Cat(name=wangwuCat), dog=Dog(name=wangwuDog))
```
***到目前为止的1/2/3点都是没毛病的，都属于Spring的正常用法***
***但是下面第4点开始，将让Cat/Dog的bean的id不与Person类中的属性名相同，并同时使用P命名空间与C命名空间***
# 4、搞事情
**Spring配置：**
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        https://www.springframework.org/schema/context/spring-context.xsd">

	<!--    启动自动装配-->
    <context:annotation-config />

	<--这里的id（cat和dog）与Person类中的属性名cat和dog一致-->
    <bean id="cat" class="cn.BinGCU.pojo.AninalImpl.Cat" p:name="zhangsanCat"/>
    <bean id="dog" class="cn.BinGCU.pojo.AninalImpl.Dog" p:name="zhangsanDog"/>

	<--这里的id（catttttttt和doggggg）与Person类中的属性名cat和dog不一致-->
	<bean id="catttttttt" class="cn.BinGCU.pojo.AninalImpl.Cat"  p:name="wangwuCat"/>
    <bean id="doggggg" class="cn.BinGCU.pojo.AninalImpl.Dog"  p:name="wangwuDog"/>

	<--这里指定了id为catttttttt的引用，而不是id为cat的引用-->
	<--那么person的两只宠物到底是叫zhangsanCat，zhangsanDog还是wangwuCat，wangwuDog呢？-->
	<bean id="person01" class="cn.BinGCU.pojo.Person" p:cat-ref="catttttttt" p:dog-ref="doggggg"/>
    <bean id="person02" class="cn.BinGCU.pojo.Person" c:cat-ref="catttttttt" c:dog-ref="doggggg"/>
</beans>

```

**Test：**
```java
	ApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");
	
    Person person01 =  context.getBean("person01 ", Person.class);
    person01.
    System.out.println("P命名空间：");
    System.out.println(person01 );
	
	Person person02 = context.getBean("person02", Person.class);
	person02.managePets();
	System.out.println("C命名空间：");
	System.out.println(person02);
```
**Output**
```output
Manage Dog and Cat
Cat: miao~
Dog: bark~
P命名空间：
Person(cat=Cat(name=zhangsanCat), dog=Dog(name=zhangsanDog))

Manage Dog and Cat
Cat: miao~
Dog: bark~
C命名空间：
Person(cat=Cat(name=wangwuCat), dog=Dog(name=wangwuDog))
```
**瞧，person02的情况是两只宠物是属于wangwu的，所以自动装配的DI（dependency injection）优先级高于在同一个bean标签下的c命名空间的DI优先级。
而person01的情况，也反映了P命名空间的DI优先级大于自动装配，因为它能不受autowiring影响而成功找到对应的类型引用**

***
**所以说在采用自动装配的一个bean配置文件中同时使用p/c命名空间，从安全角度，如果你在没有搞清楚具体能不能这样注入值的情况下这么做，那么此般举动将是危险的。当Spring所自动装配的bean中，有某个属性默认注入值为null，在复杂的业务中，那同样也会引发大问题，更别提实际业务中的bug分析有多么令人头疼了**
**所以不要想着方便，而在一个需要手动注入、却只有一部分需要自动装配的bean配置中开启 <context:annotation-config />，因为在这种情况c命名空间将被自动装配无效化**