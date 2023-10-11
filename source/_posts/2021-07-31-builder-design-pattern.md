---
title: 手写基于编译期的建造者模式实体类生成器
date: 2021-07-31 22:19
tags:
- Java
---

# 背景
今天我们来聊聊建造者模式，对于建造者模式的理论和一些描述代码网上已经有非常多的文章了，在这里也就不重复赘述了，所以今天来聊聊不一样的。
[Lombok](https://projectlombok.org/features/Builder)想必大家都听说过，就是通过注解，在编译期间修改语法树，最后javac再将修改后的语法树编译成class文件。
Lombok有一个@Builder注解，其作用是为添加了该注解的类生成建造者模式的api，例如有以下类
<!--more-->
```java
@Builder
public class User {
    private int id;
    private int name;
}
```
只需要在类上方添加@Builder注解，就可以像下面这样创建User对象
```java
User user = User.builder()
    .id(1)
    .name("name").build();
```
这样的api用起来比传入参数进构造器和手动setXXX优雅多了，当然Lombok在背后都做了什么我们不得而知，感兴趣的同学可以深入了解一下。
我们今天要模仿Lombok的api手写一个带Builder模式的实体类，
```java
public class User {
    private int id;
    private String name;
    private String sex;
    private String address;

    // 构造器中需要为所有属性设置默认值，这步很重要，不然下面的set可能会出现NPE
    // 同时构造方法设置为私有，防止被外部私自实例化对象
    private User() {
        id = 0;
        name = "";
        sex = "";
        address = "";
    }

    // 静态内部建造类，所有对对象属性的操作均由该类完成，同时建造类的构造方法也为私有，目的同上
    // setXX为设置属性值，clearXX为恢复属性默认值，setXX和clearXX会返回Builder自己，所以可以做到链式调用
    public static class Builder {
        private User user;
        private Builder (User user) {
            this.user = user;
        }

        public Builder setId(int id) {
            user.id = id;
            return this;
        }

        public Builder setName(String name) {
            user.name = name;
            return this;
        }

        public Builder setSex(String sex) {
            user.sex = sex;
            return this;
        }

        public Builder setAddress(String address) {
            user.address = address;
            return this;
        }

        public Builder clearId() {
            user.id = 0;
            return this;
        }

        public Builder clearName() {
            user.name = "";
            return this;
        }

        public Builder clearAddress() {
            user.address = "";
            return this;
        }

        public Builder clearSex() {
            user.sex = "";
            return this;
        }

        // 返回构建好的对象
        public User build() {
            return user;
        }
    }

    // 创建建造者对象
    public static Builder newBuilder() {
        return new Builder(new User());
    }

    // 将当前对象转为建造者对象
    public Builder toBuilder() {
        return new Builder(this);
    }

    public int getId() {
        return id;
    }

    public String getName() {
        return name;
    }

    public String getSex() {
        return sex;
    }

    public String getAddress() {
        return address;
    }

    @Override
    public String toString() {
        return "User{" +
            "id=" + id +
            ", name='" + name + '\'' +
            ", sex='" + sex + '\'' +
            ", address='" + address + '\'' +
            '}';
    }
}
```
用法如下
```java
User user = User.newBuilder()
    .setId(1)
    .setName("name").build();

user = user.toBuilder()
    .clearName().build();
```
可以看到我们手写的类功能比Lombok的还要丰富，不仅可以设置对象属性的值，还可以清除对象属性的值，甚至可以将建造者和对象互相转换。但是写这样一个类工作量太大，
一个项目中往往有几十个类，如果每个类都这么写，那其他事都干不了了，所以借鉴Lombok的思想，我们通过代码生成这样的类不就解放了吗。

工具思路是通过一个json描述文件描述一个类的信息，然后使用[JavaPoet](https://github.com/square/javapoet)生成一个Java文件。（注.JavaPoet是一个通过api组装Java源代码，最后生成Java文件的lib）
核心生成代码如下
```java
public void generate(String json) throws IOException {
    Gson gson = new Gson();
    Entity entity = gson.fromJson(json, Entity.class);
    String entityClassName = WordUtils.capitalize(entity.getName());
    String entityObjectLiteral = WordUtils.uncapitalize(entityClassName);
    ClassName className = ClassName.get(entity.getPackageName(), entityClassName);

    TypeSpec.Builder entityTypeBuilder = TypeSpec.classBuilder(entityClassName)
        .addModifiers(Modifier.PUBLIC);
    MethodSpec entityConstructor = MethodSpec.constructorBuilder()
        .addModifiers(Modifier.PRIVATE).build();
    entityTypeBuilder.addMethod(entityConstructor);

    TypeSpec.Builder builderTypeBuilder = TypeSpec.classBuilder("Builder")
        .addModifiers(Modifier.PUBLIC, Modifier.STATIC)
        .addField(className, entityObjectLiteral, Modifier.PRIVATE);
    MethodSpec builderConstructor = MethodSpec.constructorBuilder()
        .addParameter(className, entityObjectLiteral)
        .addStatement("this.$N = $N", entityObjectLiteral, entityObjectLiteral)
        .addModifiers(Modifier.PRIVATE).build();
    builderTypeBuilder.addMethod(builderConstructor);

    entity.getFields().forEach(it -> {
        FieldTypeFormatAndValue fieldTypeFormatAndValue = fieldNameWithInitializeValue.get(it.getType());
        FieldSpec fieldSpec = FieldSpec.builder(fieldTypeNameWithTypeName.get(it.getType()), it.getName(), Modifier.PRIVATE)
            .initializer(fieldTypeFormatAndValue.getFormat(), fieldTypeFormatAndValue.getValue()).build();
        entityTypeBuilder.addField(fieldSpec);
        MethodSpec setMethod = MethodSpec.methodBuilder(String.format("set%s", WordUtils.capitalize(it.getName())))
            .addModifiers(Modifier.PUBLIC)
            .addParameter(fieldTypeNameWithTypeName.get(it.getType()), it.getName())
            .returns(ClassName.get("", "Builder"))
            .addStatement("$N.$N = $N", entityObjectLiteral, it.getName(), it.getName())
            .addStatement("return this").build();

        builderTypeBuilder.addMethod(setMethod);
        MethodSpec clearMethod = MethodSpec.methodBuilder(String.format("clear%s", WordUtils.capitalize(it.getName())))
            .addModifiers(Modifier.PUBLIC)
            .returns(ClassName.get("", "Builder"))
            .addStatement(String.format("$N.$N = %s", fieldTypeFormatAndValue.getFormat()), entityObjectLiteral, it.getName(), fieldTypeFormatAndValue.getValue())
            .addStatement("return this").build();
        builderTypeBuilder.addMethod(clearMethod);

        MethodSpec getMethod = MethodSpec.methodBuilder(String.format("get%s", WordUtils.capitalize(it.getName())))
            .addModifiers(Modifier.PUBLIC)
            .returns(fieldTypeNameWithTypeName.get(it.getType()))
            .addStatement("return $N", it.getName()).build();
        entityTypeBuilder.addMethod(getMethod);
    });
    MethodSpec buildMethod = MethodSpec.methodBuilder("build")
        .addModifiers(Modifier.PUBLIC)
        .returns(ClassName.get("", entityClassName))
        .addStatement("return $N", entityObjectLiteral).build();
    builderTypeBuilder.addMethod(buildMethod);

    MethodSpec newBuilderMethod = MethodSpec.methodBuilder("newBuilder")
        .addModifiers(Modifier.PUBLIC, Modifier.STATIC)
        .returns(ClassName.get("", "Builder"))
        .addStatement("return new $N(new $N())", "Builder", entityClassName).build();
    entityTypeBuilder.addMethod(newBuilderMethod);

    MethodSpec toBuilderMethod = MethodSpec.methodBuilder("toBuilder")
        .addModifiers(Modifier.PUBLIC)
        .returns(ClassName.get("", "Builder"))
        .addStatement("return new $N(this)", "Builder").build();
    entityTypeBuilder.addMethod(toBuilderMethod);

    entityTypeBuilder.addType(builderTypeBuilder.build());
    JavaFile javaFile = JavaFile.builder(entity.getPackageName(), entityTypeBuilder.build())
        .build();

    javaFile.writeToFile(new File(String.format("src/main/java")));
}
```
给定一段json
```json
{
  "name": "student",
  "packageName": "com.zshnb.patterndesign.builder",
  "fields": [
    {
      "name": "id",
      "type": "int"
    },
    {
      "name": "name",
      "type": "String"
    }
  ]
}
```
执行完会在给定的package下生成Java文件，然后我们运行测试看一下结果
![](img1.png)
![](img2.png)
测试通过，说明我们生成的Java文件跟上面手写的类结构一致，当然这个生成器可以加入更多功能，比如为List类型的属性生成addXXX以及addAllXXX等方法，感兴趣的同学可以自行扩展一下。
以及这里的读取是通过JSON，也可以通过读取Typescript的类型文件，或者protobuf的定义文件，生成最终的Java类
最后贴上项目的[github地址](https://github.com/zshnb/new-pattern-design)
