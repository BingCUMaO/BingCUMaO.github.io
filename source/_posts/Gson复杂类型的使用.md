---
title: Gson复杂类型的使用
date: 2020-07-25 22:00:00
categories:
- 技术博客

tags:
- Java
- Gson



---



# 一、类型

**Gson识别 4 种类型**

1. JsonPrimitive：其中包括基本类型的包装类和String类
2. JsonObject：类似某个具体的类型，只不过以Json格式进行存储，因此通过对应的get可以获取以Json形式存在的那些属性。
3. JsonArray：由JsonElement类型组成的数组类型，注意的是，由于是以Json格式存储的，所以JsonArray所存储的具体类型可以是混合的，为什么数组可以保存多个类型？因为存储的是Json格式的字符串。
4. JsonNull：存放Null值。

**使用**

- Gson重载了很多方法，因此Gson的使用基本都是围绕着`fromJson(...)`和`toJson(...)`。看字面意思就知道一个从Json转化到具体的类型，一个用来转为成Json格式。



# 二、泛型集合类型

集合类型，用的最多的其实是`List<>`。

- 当我们有以下这个类型的时候，我们会发现里面有`List<String>`和`List<Integer>`两个类型：

```java
package top.bingcu.gson;

import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;

import java.util.List;

@Data
@AllArgsConstructor
@NoArgsConstructor
public class BasicInfo {
    private List<String> strList;
    private List<Integer> intList;
}
```

或许你可以说使用`gson.fromJson(json, List<String>.class);`和`gson.fromJson(json, List<Integer>.class);`，但别忘了，Java的泛型可是采用了`类型擦除`机制的啊，`List<String>.class`和`List<Integer>.class`最后都会被擦除成`List.class`这个类型。所以，Java泛型是java本身的缺点之一了。

因此，Gson为我们提供了一个`TypeToken<T>`类型，专门用来解决java泛型擦除的问题。即这样使用：

```java
gson.fromJson(json, new TypeToken<List<String>>(){}.getType());
```

- **注意：**
- 这里使用的是`new TypeToken<T>(){}`，而不是`new TypeToken<T>()`。这里使用了`匿名内部类`，这样做的好处就是，我们传入了一个`T`类型，相当于我们定义了一个具体关于用户传进来的`T`类型参数的TypeToken类。
- 说人话就是我们通过匿名内部类，将`TypeToken<T>`这个父类转化为具体的、不会被java进行`类型擦除`的`TypeToken<String>`子类型，前者jvm会采取`类型擦除`行为，而后者规避了`类型擦除`。因为在子类继承父类的过程中，`T`类型已经从不确定类型变成了具体的`String`类型了。



# 三、嵌套类型

假如我们有这么一个类型：

```java
package top.bingcu.gson;

import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;


@Data
@NoArgsConstructor
@AllArgsConstructor
public class Basic {
    private BasicInfo basicInfo;
}

@Data
@NoArgsConstructor
@AllArgsConstructor
class BasicInfo {
    private BasicInfoDetail basicInfoDetail
    private List<String> strList;
    private List<Integer> intList;
}

@Data
@NoArgsConstructor
@AllArgsConstructor
class BasicInfoDetail{
    String datas;
}
```



由于`Basic`中包含`BasicInfo`，`BasicInfo`又包含两个泛型集合类型和`BasicInfoDetail`类型，因此，Gson的`toJson(...)`和`fromJson(...)`方法是无法自动识别这个如此复杂的自定义嵌套类型的。



所以我们要做的就是让Gson能够识别，需要使用到Gson提供的`JsonDeserializer`这个类了，我们可以实现这个类的`deserialize`方法使得Gson在解析`Basic`这个类的时候能够自动跑到我们写的`JsonDeserializer`实现类当中进行解析。

```java
private static  class BasicInfoDeserializer implements JsonDeserializer<BasicInfo>{

    /**
     *	这样每次解析BasicInfo类型的时候，都会自动调用该方法
     *	同时fromJson()方法返回的BasicInfo类型就都是我们所继承实现的这个方法所返回的对象
     */
    @Override
    public BasicInfo deserialize(JsonElement json, Type typeOfT, JsonDeserializationContext context) throws JsonParseException {
        Gson gson = new Gson();
        JsonObject jsonObject = json.getAsJsonObject();

        
        List<String> strList = gson.fromJson(jsonObject.get("strList").getAsString(), new TypeToken<List<String>>(){}.getType());
        List<Integer> intList = gson.fromJson(jsonObject.get("intList").getAsString(), new TypeToken<List<Integer>>(){}.getType());
        
        //设置需要返回的自定义BasicInfo类型
        BasicInfo basicInfo = new BasicInfo();
        basicInfo.setStrList(strList);
        basicInfo.setIntList(intList);
        //由于BasicInfoDetail类型里面只有一个String类型的属性，因此Gson自己能够解析的出来BasicInfoDetail类型
        basicInfo.setBasicInfoDetail = gson.fromJson(jsonObject.get("basicInfoDetail").getAsString(), BasicInfoDetail.class);

        return basicInfo;
    }
}
```



这样`Basic`、`BasicInfo`、`BasicInfoDetail`三个类中的后两个就能正常解析了。然后就剩下`Basic`类型了**：**





```java
private static class BasicDeserializer implements JsonDeserializer<Basic>{

    @Override
    public Basic deserialize(JsonElement json, Type typeOfT, JsonDeserializationContext context) throws JsonParseException {
        JsonObject jsonObject = json.getAsJsonObject();
        Gson gson = new Gson();

        //这里的BasicInfo类型，由于上面已经手写过它的解析过程了，所以Gson能够识别到并且知道如何将它的属性和Json文本对应起来
        BasicInfo basicInfo = gson.fromJson(jsonObject.get("basicInfo").getAsString(), BasicInfo.class);
        
        Basic basic = new Basic();
        basic.setBasicInfo(basicInfo);

        return basic;
    }
}
```





**这样，我们就完成了一连环嵌套类的解析过程了，之后我们便可以直接解析Basic类了：**

```java
Gson gson = new Gson();
Basic basic = gson.fromJson(json, Basic.class);
```

