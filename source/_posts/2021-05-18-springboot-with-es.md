---
layout: post
title: SpringBoot配置ElasticsearchRestTemplate
date: 2021-05-18 17:17
tags:
- Java
- SpringBoot
- Elasticsearch

---
# 背景
最近项目中用到了Elasticsearch，需要在SpringBoot项目上配置，网上找了一圈发现都是使用`ElasticsearchTemplate`操作，官方最新的推荐是使用
`ElasticsearchRestTemplate`，基于HTTP协议与es交互。于是各种查资料，踩坑，在这里把一步步配置的过程记录一下。

# 集成
1. 首先创建一个SpringBoot项目，添加最基本的依赖和es的依赖(SpringBoot版本为2.3.3)
    ```xml

    <dependency>
        <artifactId>spring-boot-starter-data-elasticsearch</artifactId>
        <groupId>org.springframework.boot</groupId>
    </dependency>
    ```

2. 创建es配置Bean类
    ```java

    @Configuration
    public class ESConfig {
        @Bean
        public ElasticsearchRestTemplate elasticsearchRestTemplate(RestHighLevelClient client) {
            return new ElasticsearchRestTemplate(client);
        }
    }
    ```
3. 在配置文件添加es的配置
    ```yaml
    spring:
      elasticsearch:
        rest:
          uris: http://ip:9200
    ```
4. 创建es的实体类
    ```java
    @Document(indexName = "test-user", createIndex = false)
    public class UserForSearch {
        @Id
        private int id;
        @Field(type = FieldType.Keyword)
        private String nickname;

        public int getId() {
            return id;
        }

        public void setId(int id) {
            this.id = id;
        }

        public String getNickname() {
            return nickname;
        }

        public void setNickname(String nickname) {
            this.nickname = nickname;
        }

        @Override
        public String toString() {
            return "UserForSearch{" +
                "id=" + id +
                ", nickname='" + nickname + '\'' +
                '}';
        }
    }
    ```   
5. 使用`ElasticsearchTemplate`添加，查询数据
    ```java

    @Component
    class EsService {
        @Autowired
        private ElasticsearchRestTemplate elasticsearchRestTemplate;

        public void testEs() {
            elasticsearchRestTemplate.delete("1", UserForSearch.class);
            UserForSearch userForSearch = elasticsearchRestTemplate.get("1", UserForSearch.class);
            System.out.println(userForSearch);

            userForSearch = new UserForSearch();
            userForSearch.setId(1);
            userForSearch.setNickname("name");
            elasticsearchRestTemplate.save(userForSearch);
            userForSearch = elasticsearchRestTemplate.get("1", UserForSearch.class);

            System.out.println(userForSearch);
        }
    }
    ```
运行结果如下
```
null
UserForSearch{id=1, nickname='name'}
 ```

第一次查询没有数据，输出null，然后调用save方法保存一条数据，接着查询并打印输出，刚才添加的数据成功显示。这样SpringBoot和es的配置就完成了，后面的
使用Repository也都大同小异，查找一下资料即可。
