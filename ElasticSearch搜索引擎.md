## ElasticSearch搜索引擎  
支持分布式、海量数据实时存储与检索技的搜索引擎技术，主要用于海量数据的分析场景  
## 数据格式  
![result](https://static01.imgkr.com/temp/d53f2fa98cd14130b7ee94deb473b9bf.jpg)  
 ES里的Index可以看做数据库的一个库，Documents则相当于数据库中表的行。  
 ## 索引  
 1、PUT请求具有幂等性(每次操作的结果都相同)，每次PUT请求创建的结果都是一样的，再次请求时，由于ES中已经存在名为work的索引了，所以会创建失败。  
 2、POST是不具有幂等性的，所以POST请求后，每次结果不一样，所以添加索引的时候是不允许使用POST请求的。  
