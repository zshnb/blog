---
title: Jackson序列化有继承关系的类属性消失
date: 2025-08-02 17:04:13
tags:
  - SpringBoot
---

最近在做一个有关答题系统的需求，前端同事希望能使用JSON Form和JSON Schema作为数据的交互和答案的验证，于是对后端来说就需要将JSON和类互相转换。由于题目有单选和多选两种类型，相应的JSON也有不同的结构如下
<!--more-->
```
// 单选
"NpBx9tQ8YH2vFg3JrLkZu": {
  "type": "string",
  "enum": [
    "0",
    "1"
  ]
},

// 多选
"qN5KrJ8XzLp7YvW2Tm9QC": {
  "type": "array",
  "uniqueItems": true,
  "minItems": 1,
  "items": {
    "type": "string",
    "enum": [
      "0",
      "1",
      "2",
      "3"
    ]
  }
},

```
对应Java类如下：

```
@JsonTypeInfo(
            use = JsonTypeInfo.Id.NAME,
            include = JsonTypeInfo.As.EXISTING_PROPERTY,
            property = "type")
@JsonSubTypes({
        @JsonSubTypes.Type(value = RadioQuiz.class, name = "string"),
        @JsonSubTypes.Type(value = CheckboxQuiz.class, name = "array"),
})
public abstract static class Quiz {
        private String type;
}

@EqualsAndHashCode(callSuper = true)
@Data
public static class RadioQuiz extends Quiz {
    @JsonProperty("enum")
    private List<String> enums;
}

@EqualsAndHashCode(callSuper = true)
@Data
public static class CheckboxQuiz extends Quiz{
    private boolean uniqueItems;
    private int minItems;
    private CheckboxQuizItem items;
}

@Data
public static class CheckboxQuizItem {
    private String type;
    @JsonProperty("enum")
    private List<String> enums;
}

```
通过Jackson的@JsonTypeInfo和@JsonSubTypes做到把不同子结构的json对象和Java类互相转换（对Jackson的这种转换不太熟悉的可以看[这篇文章](https://blog.gaoyuexiang.cn/jackson-inheritance/#class-%E6%A0%87%E8%AF%86)）。结果运行后发现序列化之后的json里没有type属性的值。

![](img1.webp)


经过一番搜索，从stackoverflow找到一个[回答](https://stackoverflow.com/questions/33611199/jackson-jsontypeinfo-property-is-being-mapped-as-null)，需要在`JsonTypeInfo`注解里加上`visible=true` ，重新运行结果如下：

![](img2.webp)



