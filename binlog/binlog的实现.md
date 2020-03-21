# binlog的实现

## 事件类型
> 代码地址 sql/log_event.h sql/log_event.cc ， log_event.cc 中有一个事件code和文字的映射表，截取下来就是事件类型。
>
``` C++

const char* Log_event::get_type_str(Log_event_type type)
{
  switch(type) {
  case START_EVENT_V3:  return "Start_v3";
  case STOP_EVENT:   return "Stop";
  case QUERY_EVENT:  return "Query";
  case ROTATE_EVENT: return "Rotate";
  case INTVAR_EVENT: return "Intvar";
  case LOAD_EVENT:   return "Load";
  case NEW_LOAD_EVENT:   return "New_load";
  case CREATE_FILE_EVENT: return "Create_file";
  case APPEND_BLOCK_EVENT: return "Append_block";
  case DELETE_FILE_EVENT: return "Delete_file";
  case EXEC_LOAD_EVENT: return "Exec_load";
  case RAND_EVENT: return "RAND";
  case XID_EVENT: return "Xid";
  case USER_VAR_EVENT: return "User var";
  case FORMAT_DESCRIPTION_EVENT: return "Format_desc";
  case TABLE_MAP_EVENT: return "Table_map";
  case PRE_GA_WRITE_ROWS_EVENT: return "Write_rows_event_old";
  case PRE_GA_UPDATE_ROWS_EVENT: return "Update_rows_event_old";
  case PRE_GA_DELETE_ROWS_EVENT: return "Delete_rows_event_old";
  case WRITE_ROWS_EVENT_V1: return "Write_rows_v1";
  case UPDATE_ROWS_EVENT_V1: return "Update_rows_v1";
  case DELETE_ROWS_EVENT_V1: return "Delete_rows_v1";
  case BEGIN_LOAD_QUERY_EVENT: return "Begin_load_query";
  case EXECUTE_LOAD_QUERY_EVENT: return "Execute_load_query";
  case INCIDENT_EVENT: return "Incident";
  case IGNORABLE_LOG_EVENT: return "Ignorable";
  case ROWS_QUERY_LOG_EVENT: return "Rows_query";
  case WRITE_ROWS_EVENT: return "Write_rows";
  case UPDATE_ROWS_EVENT: return "Update_rows";
  case DELETE_ROWS_EVENT: return "Delete_rows";
  case GTID_LOG_EVENT: return "Gtid";
  case ANONYMOUS_GTID_LOG_EVENT: return "Anonymous_Gtid";
  case PREVIOUS_GTIDS_LOG_EVENT: return "Previous_gtids";
  case HEARTBEAT_LOG_EVENT: return "Heartbeat";
  default: return "Unknown";				/* impossible */
  }
}

```

> 举例：一次写入事件，会发生QUERY_EVENT、TABLE_MAP_EVENT、WRITE_ROWS_EVENT、XID_EVENT。QUERY_EVENT是
> 事务开始记录的事件，内容就是BEGIN。TABLE_MAP_EVENT是描述即将变化的表的事件。最后是WRITE_ROWS_EVENT表示插入了一条数据。
>

## Log_event 结构

> 文件在sql/log_event.h
>
>
> Log_event的二进制结构有三部分内容Common-Header、Post-Header、Body。
> * 通用头部
>   > 在同一个mysql版本中，所有的log_event 有相同的头部信息。例如每个头部字段的含义和所占用的长度
> * post-header
>   > 每种log的特殊头部信息的格式。
> * body 
>   > 保存的信息。                  

<table>
  <caption>Common-Header 通用头部</caption>
  <tr>
    <th>Name 名称</th>
    <th>Format 格式</th>
    <th>Description 描述</th>
  </tr>
  <tr>
    <td>timestamp</td>
    <td>4 byte unsigned integer</td>
    <td>The time when the query started, in seconds since 1970.
    </td>
  </tr>
  <tr>
    <td>type</td>
    <td>1 byte enumeration</td>
    <td>See enum #Log_event_type.</td>
  </tr>
  <tr>
    <td>server_id</td>
    <td>4 byte unsigned integer</td>
    <td>Server ID of the server that created the event.</td>
  </tr>
  <tr>
    <td>total_size</td>
    <td>4 byte unsigned integer</td>
    <td>The total size of this event, in bytes.  In other words, this
    is the sum of the sizes of Common-Header, Post-Header, and Body.
    包括了通用头、post-header和body的所有信息占用的字节数量
    </td>
  </tr>
  <tr>
    <td>master_position</td>
    <td>4 byte unsigned integer</td>
    <td>The position of the next event in the master binary log, in
    bytes from the beginning of the file.  In a binlog that is not a
    relay log, this is just the position of the next event, in bytes
    from the beginning of the file.  In a relay log, this is
    the position of the next event in the master's binlog.
    在binlog中保存的是下一个事件的地址，在relaylog中保存的是master的binlog地址
    </td>
  </tr>
  <tr>
    <td>flags</td>
    <td>2 byte bitfield</td>
    <td>See Log_event::flags.</td>
  </tr>
 </table>                       
 
 
 ## binlog的写入过程
 
 