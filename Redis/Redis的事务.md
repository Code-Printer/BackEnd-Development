# Redis的事务  
Redis的事务的存储结构是一个先进先出的队列，将一系列将要执行的指令包装成一个整体，当执行事务时，一次性的将命令按照添加顺序执行，中间不会被干扰或中断。  
如果不开启事务，两个客户端操作同一个服务器时就会发生错误，如下图所示：  
![result](https://static01.imgkr.com/temp/be0097d0899643a08217c414670c7d72.png)  
## 基本操作  
1、开启事务：multi  
执行此指令后，后续的指令都会被添加到事务队列中等待执行，添加到队列中并不会立即执行，只有执行事务时命令才会被执行。  
2、执行事务：exec  
执行此指令后，事务队列中的任务会被依次执行，与multi成对使用。
演示:  
![result](https://static01.imgkr.com/temp/ca3958742ef54c3999f5e674d7b083a5.png)  
3、取消事务：discard  
终止当前事务的定义（发生在exec之前，也就是说还没有执行事务时终止事务）。 
![result](https://static01.imgkr.com/temp/b55a5823a79a433f9d340d4abba1b0f7.png)  
## Redis的指令执行流程  
![result](https://static01.imgkr.com/temp/5098bd9f636144b88364b8a326058221.png)    
## Jedis事务操作实例  
```
public class TestAffairs {
    public static void main(String[] args) {
        Jedis jedis = new Jedis("127.0.0.1",6379);
        JSONObject jsonObject = new JSONObject();  //json对象的使用
        jsonObject.put("hello","word");
        jsonObject.put("name","mrggz");
        String result = jsonObject.toJSONString();  //将json对象转换成字符串
        Transaction multi = jedis.multi(); //开启事务
        try{
            multi.set("user1",result);  //在事务中添加要执行命令
            multi.set("user2",result);
//            int res = 1 / 0;
            multi.exec();  //执行该事务
        }catch (Exception e){
            multi.discard();  //放弃该事务
            e.printStackTrace();
        }finally {
            System.out.println(jedis.get("user1"));
            System.out.println(jedis.get("user2"));
            jedis.close();  //关闭redis连接
        }
    }
}
```
## 注意事项  
### 编译型命令错误  
整个事务的所有命令都不会执行。  
### 运行型语法错误异常(例：对非整型数据执行incr操作)  
正确的命令会被执行，运行错误的指令不会执行（错误命令之后的正确指令也会被执行），已经执行完毕的命令对应的数据不会回滚，需要手动实现回滚。    
## redis使用watch实现对多线程事务操作的对象数据加乐观锁   
```
watch key //对指定key加乐观锁(相当于获取加锁数据的version值)，保持多线程事务中key的一致性    
unwatch key //当多线程事务执行失败，需要对加锁的key解锁，再加锁才可以获取到最新key的version值
```  


