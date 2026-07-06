# Java 操作 Redis

## 一、环境准备
### 1、引入依赖
```xml
<!--引入jedis连接依赖-->
<dependency>
  <groupId>redis.clients</groupId>
  <artifactId>jedis</artifactId>
  <version>2.9.0</version>
</dependency>
```

### 2、创建jedis对象
```java
 public static void main(String[] args) {
   //1.创建jedis对象
   Jedis jedis = new Jedis("192.168.40.4", 6379);//1.redis服务必须关闭防火墙  2.redis服务必须开启远程连接
   jedis.select(0);//选择操作的库默认0号库
   //2.执行相关操作
   //....
   //3.释放资源
   jedis.close();
 }
```

## 二、操作key相关API
```java
private Jedis jedis;
    @Before
    public void before(){
        this.jedis = new Jedis("192.168.202.205", 6379);
    }
    @After
    public void after(){
        jedis.close();
    }

    //测试key相关
    @Test
    public void testKeys(){
        //删除一个key
        jedis.del("name");
        //删除多个key
        jedis.del("name","age");

        //判断一个key是否存在exits
        Boolean name = jedis.exists("name");
        System.out.println(name);

        //设置一个key超时时间 expire pexpire
        Long age = jedis.expire("age", 100);
        System.out.println(age);

        //获取一个key超时时间 ttl
        Long age1 = jedis.ttl("newage");
        System.out.println(age1);

        //随机获取一个key
        String s = jedis.randomKey();

        //修改key名称
        jedis.rename("age","newage");

        //查看可以对应值的类型
        String name1 = jedis.type("name");
        System.out.println(name1);
        String maps = jedis.type("maps");
        System.out.println(maps);
    }
```

## 三、操作String相关API
```java
//测试String相关
    @Test
    public void testString(){
        //set
        jedis.set("name","小陈");
        //get
        String s = jedis.get("name");
        System.out.println(s);
        //mset
        jedis.mset("content","好人","address","海淀区");
        //mget
        List<String> mget = jedis.mget("name", "content", "address");
        mget.forEach(v-> System.out.println("v = " + v));
        //getset
        String set = jedis.getSet("name", "小明");
        System.out.println(set);

        //............
    }
```

## 四、操作List相关API
```java
//测试List相关
    @Test
    public void testList(){

        //lpush
        jedis.lpush("names1","张三","王五","赵柳","win7");

        //rpush
        jedis.rpush("names1","xiaomingming");

        //lrange

        List<String> names1 = jedis.lrange("names1", 0, -1);
        names1.forEach(name-> System.out.println("name = " + name));

        //lpop rpop
        String names11 = jedis.lpop("names1");
        System.out.println(names11);

        //llen
        jedis.linsert("lists", BinaryClient.LIST_POSITION.BEFORE,"xiaohei","xiaobai");

      	//........

    }
```

## 五、操作Set的相关API
```java
//测试SET相关
@Test
public void testSet(){

  //sadd
  jedis.sadd("names","zhangsan","lisi");

  //smembers
  jedis.smembers("names");

  //sismember
  jedis.sismember("names","xiaochen");

  //...
}
```

## 六、操作ZSet相关API
```java
//测试ZSET相关
@Test
public void testZset(){

  //zadd
  jedis.zadd("names",10,"张三");

  //zrange
  jedis.zrange("names",0,-1);

  //zcard
  jedis.zcard("names");

  //zrangeByScore
  jedis.zrangeByScore("names","0","100",0,5);

  //..

}
```

## 七、操作Hash相关API
```java
//测试HASH相关
@Test
public void testHash(){
  //hset
  jedis.hset("maps","name","zhangsan");
  //hget
  jedis.hget("maps","name");
  //hgetall
  jedis.hgetAll("mps");
  //hkeys
  jedis.hkeys("maps");
  //hvals
  jedis.hvals("maps");
  //....
}
```



> 更新: 2022-06-17 14:29:17  
> 原文: <https://www.yuque.com/thinkspace/lcb0zg/mrpzc4>