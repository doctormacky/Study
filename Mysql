#1. 如何优化 mysql 中 模糊查询 逻辑

##前匹配： 使用用 like 'prefix%' 的形式，这种情况Mysql可以正常利用索引
```sql
select * from users where username like 'John%'
```

##后匹配： 对于需要用like '%suffix' 的例子，可以创建一个辅助列存储反转字符串，基于这个列来匹配
```sql
alter table users add reversed_username  VARCHAR(255);
update users set reversed_username = REVERSE(username);
create index idx_reversed_username on users(reversed_username);
```

这个时候，如果存储的事John Macky， 那么reversed_username 会是 ykcmM nhoJ
那么sql 会这么写 select * from users where reversed_username like 'nhoJ%'

尽量添加更多的条件缩小范围，例如
```sql
select * from users where created_at >= '2024-10-10' and username like 'John%';
```
