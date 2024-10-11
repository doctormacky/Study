## 如何优化mysql中模糊查询 逻辑

### 前匹配:使用用 like 'prefix%' 的形式，这种情况Mysql可以正常利用索引
```sql
select * from users where username like 'John%'
```

### 后匹配:对于需要用like '%suffix' 的例子，可以创建一个辅助列存储反转字符串，基于这个列来匹配
```sql
alter table users add reversed_username  VARCHAR(255);
update users set reversed_username = REVERSE(username);
create index idx_reversed_username on users(reversed_username);
```

这个时候，如果存储的事John Macky， 那么reversed_username 会是 `ykcmM nhoJ`， 那么sql会这么写 
```
select * from users where reversed_username like 'nhoJ%'
```

### 尽量添加更多的条件缩小范围，例如
```sql
select * from users where created_at >= '2024-10-10' and username like 'John%';
```

## sql用了函数后，索引一定会失效吗

### 对索引列使用函数：
  如果在where 子句中对索引列应用了函数转换，例如lower(column_name) 会导致索引无法被使用，这是因为索引是基于列的原始值而构建的，函数改变了这些值
  例如
  ```sql
    select * from users where lower(usernmae) = 'john';
  ```
### 计算表达式
  如果计算表达式例如column + 1 可能导致mysql 放弃使用索引
  例如
  ```sql
  select * from orders where price + 10 > 100;
  ```

### 不会导致索引失效的情况

#### 函数在索引范围之外使用
  函数不在索引范围（即where 或者 on 条件）使用的时候，索引依然有效， 例如在`select` 列表，`order by` `group by` 等地方使用函数通常不影响索引的使用
  例如
  ```
  select upper(username) where username = 'John';
  ```

### 建立额外的字段，然后对该字段建新的索引
