# Spring Boot 整合 Redis

## 一、简介

Spring Boot Data Redis 中提供了`RedisTemplate`和`StringRedisTemplate`，其中`StringRedisTemplate`是`RedisTemplate`的子类，两个方法基本一致，不同之处主要体现在操作的数据类型不同，`RedisTemplate`中的两个泛型都是`Object`，意味着存储的`key`和`value`都可以是一个对象，而`StringRedisTemplate`的两个泛型都是`String`，意味着`StringRedisTemplate`的`key`和`value`都只能是字符串。

> 注意: 使用`RedisTemplate`默认是将对象序列化到`Redis`中,所以放入的对象必须实现对象序列化接口

## 二、环境准备

### 1、引入依赖

```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
```

### 2、添加配置

```properties
spring.redis.host=localhost
spring.redis.port=6379
spring.redis.database=0
```

## 三、实战

```java
	@Autowired
    private StringRedisTemplate stringRedisTemplate;  //对字符串支持比较友好,不能存储对象
    @Autowired
    private RedisTemplate redisTemplate;  //存储对象

    @Test
    public void testRedisTemplate(){
        
        //设置redistemplate值使用对象序列化策略
        redisTemplate.setValueSerializer(new JdkSerializationRedisSerializer());//指定值使用对象序列化
        
        //redisTemplate.opsForValue().set("user",new User("21","小黑",23,new Date()));
        User user = (User) redisTemplate.opsForValue().get("user");
        System.out.println(user);
        
//      Set keys = redisTemplate.keys("*");
//      keys.forEach(key -> System.out.println(key));
        
        /*redisTemplate.opsForValue().set("name","xxxx");
        Object name = redisTemplate.opsForValue().get("name");
        System.out.println(name);*/
        
        /*redisTemplate.opsForList().leftPushAll("lists","xxxx","1111");
        List lists = redisTemplate.opsForList().range("lists", 0, -1);
        lists.forEach(list-> System.out.println(list));*/
    }


    //key的绑定操作 如果日后对某一个key的操作及其频繁,可以将这个key绑定到对应redistemplate中,日后基于绑定操作都是操作这个key
    //boundValueOps 用来对String值绑定key
    //boundListOps 用来对List值绑定key
    //boundSetOps 用来对Set值绑定key
    //boundZsetOps 用来对Zset值绑定key
    //boundHashOps 用来对Hash值绑定key

    @Test
    public void testBoundKey(){
        BoundValueOperations<String, String> nameValueOperations = stringRedisTemplate.boundValueOps("name");
        nameValueOperations.set("1");
        //yuew
        nameValueOperations.set("2");
        String s = nameValueOperations.get();
        System.out.println(s);

    }


    //hash相关操作 opsForHash
    @Test
    public void testHash(){
        stringRedisTemplate.opsForHash().put("maps","name","小黑");
        Object o = stringRedisTemplate.opsForHash().get("maps", "name");
        System.out.println(o);
    }

    //zset相关操作 opsForZSet
    @Test
    public void testZSet(){
        stringRedisTemplate.opsForZSet().add("zsets","小黑",10);
        Set<String> zsets = stringRedisTemplate.opsForZSet().range("zsets", 0, -1);
        zsets.forEach(value-> System.out.println(value));
    }

    //set相关操作 opsForSet
    @Test
    public void testSet(){
        stringRedisTemplate.opsForSet().add("sets","xiaosan","xiaosi","xiaowu");
        Set<String> sets = stringRedisTemplate.opsForSet().members("sets");
        sets.forEach(value-> System.out.println(value));
    }

    //list相关的操作opsForList
    @Test
    public void testList(){
        // stringRedisTemplate.opsForList().leftPushAll("lists","张三","李四","王五");
        List<String> lists = stringRedisTemplate.opsForList().range("lists", 0, -1);
        lists.forEach(key -> System.out.println(key));
    }


    //String相关的操作 opsForValue
    @Test
    public void testString(){
        //stringRedisTemplate.opsForValue().set("166","好同学");
        String s = stringRedisTemplate.opsForValue().get("166");
        System.out.println(s);
        Long size = stringRedisTemplate.opsForValue().size("166");
        System.out.println(size);
    }


    //key相关的操作
    @Test
    public void test(){
        Set<String> keys = stringRedisTemplate.keys("*");//查看所有key
        Boolean name = stringRedisTemplate.hasKey("name");//判断某个key是否存在
        stringRedisTemplate.delete("age");//根据指定key删除
        stringRedisTemplate.rename("","");//修改key的名称
        stringRedisTemplate.expire("key",10, TimeUnit.HOURS);
      	//设置key超时时间 参数1:设置key名 参数2:时间 参数3:时间的单位
        stringRedisTemplate.move("",1);//移动key
    }
```


> 更新: 2022-04-09 16:53:09  
> 原文: <https://www.yuque.com/thinkspace/gs6fp8/pplogg>