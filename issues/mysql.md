# mysql utfsmb4
异常日志 ： Request processing failed; nested exception is org.springframework.jdbc.UncategorizedSQLException: PreparedStatementCallback; uncategorized SQLException for SQL [insert into tbl_message_tts (time_to_send,message) values (?,?)]; SQL state [HY000]; error code [1366]; Incorrect string value: '\xF0\x9F\x99\x85\"...' for column 'message' at row 1; nested exception is java.sql.SQLException: Incorrect string value: '\xF0\x9F\x99\x85\"...' for column 'message' at row 1
## 解决：
数据需要重建以支持utf8mb4字符集，修改相关表的字符集；
ALTER TABLE `tbl_message_tts` convert to character set utf8mb4 COLLATE utf8mb4_general_ci;
程序需要加上下面语句
jdbcTemplate.execute("set names utf8mb4");
jdbcTemplate.execute("......");


# 报错
 MySql Host is blocked because of many connection errors; unblock with 'mysqladmin flush-hosts'
## 解决：flush hosts;