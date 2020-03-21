# mysql server 启动

* 入口函数
> sql/mysqld.cc  mysqld_main 

* 初始化

    *  读取命令行参数，加载配置文件。<br>
    *  init_sql_statement_names().  初始化statement的一个数组，里面放了一些sql的语句声明，可能后边要用。
    *  sys_var_init(); 初始化mysql的配置参数。
    *  handle_early_options() 。初始化一些组件，是否启用performance_schema、help和bootstrap相关配置，因为server的其他部分要依赖它们。 
    *   mysql_audit_initialize(). 审计相关功能。
    *  logger.init_base(); 初始化log的基本功能，初始化handler函数和错误日志的初始化。table log handler 不在这里初始化。
    *  my_init_signals(); 处理一些软终端信号。
        > 设置信号的处理方式：屏蔽信号 SIGSEGV（一个进程执行了一个无效的内存引用）、SIGABRT、SIGBUS、SIGILL、SIGFPE等，一些默认行为是关闭进程的信号。
        
        > 改变一些信号的默认处理方式，例如SIGPIPE、SIGQUIT、SIGHUP等。
    *   pthread_attr_setstacksize(&connection_attrib,my_thread_stack_size + guardize);分配堆栈空间大小。  
    *   生成服务器的uuid：读取auto.cnf.如果存在就不生成， 如果不存在就生成uuid后写入auto.cnf
    *   如果开启了binlog，进行一些binlog的初始化。如果开启了gtid，则进行一些工作，大概的意思是把系统启动时的一些事件写入到binlog（不太确定）。
    *   init_ssl()
    *   网络初始化。
        > 设置端口，没有配置就使用3306。ipv4、ipv6的,bind,listen
    *   mysql_rm_tmp_tables
    *   acl_init  从mysql的权限表读取用户权限
    *   my_tz_init 初始化时区？有什么用？
    *   servers_init(0);  servers_reload.
    *   udf_init()
    *   init_status_vars();初始化状态参数。可能是把配置参数读到mysql的表中。可以使用show命令查看
    *   check_binlog_cache_size check_binlog_stmt_cache_size 检查binlog的cache大小
    *   init_slave 配置relaylog，slaveio，slavesql 
    *   execute_ddl_log_recovery  从log中恢复数据
    *   Events::init(opt_noacl || opt_bootstrap) 初始化事件，事件队列，事件调度器
    *   start_handle_manager() 启动handler管理的线程
    *   handle_connections_sockets()  到这里就成功了，mysql在这里等待client连接                                                                                                                                
                                                                                                                                   
                                                                                                                                             