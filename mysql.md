## Mysql事务隔离级别

![image-20220324220150107](https://gitee.com/ingachin/mdimage/raw/master/image-20220324220150107.png)

**不可重复读**

**不可重复读表示: 事务 A 多次读取同一数据，事务 B 在事务A多次读取的过程中，对数据作了更新并提交，导致事务A多次读取同一数据时，结果 不一致。**



**幻读**

**一个事务先根据某些条件查询出一些记录，之后另一个事务又向表中插入了符合这些条件的记录，原先的事务再次按照该条件查询时，能把另一个事务插入的记录也读出来。（幻读在读未提交、读已提交、可重复读隔离级别都可能会出现）**



![image-20220324220636528](https://gitee.com/ingachin/mdimage/raw/master/image-20220324220636528.png)

## Mysql主从复制

![image-20220418210413252](https://gitee.com/ingachin/mdimage/raw/master/image-20220418210413252.png)