# binlog的概念


## binlog的作用

>    mysql的日志有很多，慢日志、错误日志、查询日志、innodb的undo和redo，binlog和relaylog。
binlog是mysql的日志，不是存储引擎的日志。mysql的诸多日志从字面就可以理解它的含义，例如慢日志记录的是运行缓慢的sql的日志，错误日志是记录mysql的
错误信息，查询日志是记录所有的查询操作（这个没什么太大用处，mysql默认也是关闭的）。而binlog记录的是什么，用来做什么的？
>
>   binlog是英文bin和log组合而成，即二进制日志。
计算机对于数据基本的组织和运算都是基于二进制的，CPU读取内存的数据也是通过高低点位来判断该位数据是0或1.
>
> 因此mysql的二进制日志在这里的意义可能是基于业务的，区别与普通的文本日志，binlog需要记录的信息是数据的变化，包括了DDL和DML，当然查询是不记录的。
binlog的主要用途就是mysql的主从复制。进入到binlog的存储目录，随便对任何binlog使用file命令，可以看到结果...MySQL replication log。
这就已经很明显了，binlog的文件类型翻译成中文就是mysql复制日志。由于binlog的特点，它也当然的可以作为一种数据恢复的方法，这方面不了解，但是理论上应该是可以的。
>我们还可以基于binlog的数据变化做数据的传输、监控异常数据等，这些已经被广泛应用了，例如mysql的增量数据同步到大数据平台。
> 
> binlog是为了服务mysql的复制机制，所以必然要对mysql的主从复制有一些了解。
>

## binlog记录的方式

> 从任何一本mysql的书籍或网上都可以得到这些信息。STATEMENT、ROW、MIXED。
>   * STATEMENT记录的是执行的语句，是数据变化的逻辑。
>   * ROW记录的是变化的结果。
>   * MIXED根据条件在STATEMENT和ROW下选择。
>   在mysql的官方文档中可以查看到在什么条件下选择ROW模式记录，例如触发器、存储过程、调用了UUID()、FOUND_ROWS()、ROW_COUNT()、USER()等等。
>
>

## 主从复制
  * 最普通的方式
>   一个master接一个slave
  * 多级复制
>  slave节点下还有slave
  
  * 多源复制
  
  > 一个slave上有多个master
>
>

## GTID 

> GTID —— Global Transaction identifier 全局事务标识符。GTID = source_id:transaction_id。
> 其中 source_id 表示源服务器。transaction_id 是序列化的数字，有顺序的数字，由事务提交的顺序决定。不同服务器的transaction_id可能相同。但是
>GTID在全局是唯一的。这个东西是做什么的已经很明显了，在全局（master和它的slave、孙slave之间，slave和它的多个master之间）每一条binlog记录都可以
>追溯其来源，并且是唯一的。那么在多源复制中就可以应用它来保障主从复制的数据一致性，因为和binlog有一些关联，所以把这个概念提出来放在这里。

