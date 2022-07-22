# ON DUPLICATE KEY UPDATE 使用总结



>   官方文档：https://dev.mysql.com/doc/refman/8.0/en/insert-on-duplicate.html
>
>   注意：`ON DUPLICATE KEY UPDATE`是 MySQL 特有的语法，使用前需要考虑清楚与其他数据库的兼容性问题



## 1、前置知识

![image-20220722132901860](markdown/ON DUPLICATE KEY UPDATE 使用总结.assets/image-20220722132901860.png)



## 2、问题引入

```mysql
CREATE TABLE `t1` (
    `a` int NOT NULL AUTO_INCREMENT,
    `b` int DEFAULT NULL,
    `c` int DEFAULT NULL,
    PRIMARY KEY (`a`)
)
```

```
You have an error in your SQL syntax; check the manual that corresponds to your MySQL server version for the right syntax to use near 'INSERT INTO t1(a,b,c) VALUES (1,1,1)' at line 2 
Duplicate entry '1' for key 't1.PRIMARY'
```

INSERT 语句指定了主键的值，因此重复执行时 MySQL 会抛出 `Duplicate 异常`



## 2、ON DUPLICATE KEY UPDATE

### 基础用法

If you specify an `ON DUPLICATE KEY UPDATE` clause and a row to be inserted would cause a duplicate value in  a `UNIQUE` index or `PRIMARY KEY`, an `UPDATE` of the old row [occurs](译：执行). For example, if column `a` is declared as `UNIQUE` and contains the value        `1`, the following two statements have **similar** effect:      

```mysql
INSERT INTO t1(a,b,c) VALUES (1,2,3)
    ON DUPLICATE KEY UPDATE c=c+1;
    
UPDATE t1 SET c=c+1 WHERE a=1;
```

>   **The effects are not quite [identical](译：完全相同的)**：For an `InnoDB` table where `a` is an auto-increment column, the `INSERT` statement increases the auto-increment value but the `UPDATE` does not. 



If column `b` is also unique, the [`INSERT`](https://dev.mysql.com/doc/refman/8.0/en/insert.html) is equivalent to this `UPDATE` statement instead:      

```mysql
UPDATE t1 SET c=c+1 WHERE a=1 OR b=2 LIMIT 1;
```

>   If `a=1 OR b=2` matches several rows, only *one* row is updated. <u>In general, you should try to avoid using an `ON DUPLICATE KEY UPDATE` clause on tables with multiple unique indexes.</u> 



If a table contains an `AUTO_INCREMENT` column  and `INSERT ... ON DUPLICATE KEY UPDATE` inserts or updates a row, the [`LAST_INSERT_ID()`](https://dev.mysql.com/doc/refman/8.0/en/information-functions.html#function_last-insert-id) function returns the `AUTO_INCREMENT` value. 

包含 `ON DUPLICATE KEY UPDATE` 子句的 `INSERT` 语句有三种返回值：

-   1：if the row is inserted as a new row
-   2：if an existing row is updated
-   0：f an existing row is set to its current values



### VALUES() 方法

The `ON DUPLICATE KEY UPDATE` clause can contain multiple column assignments, separated by commas.      

In assignment value expressions in the `ON DUPLICATE KEY UPDATE` clause, you can use the `VALUES(col_name)` function to refer to column values from the `INSERT` portion of the `INSERT ... ON DUPLICATE KEY UPDATE` statement. In other words,        `VALUES(col_name)` in the `ON DUPLICATE KEY UPDATE` clause refers to the value of *`col_name`* that would be inserted, had no duplicate-key conflict occurred. This function is especially useful in multiple-row inserts. The`VALUES()` function is meaningful        only in the `ON DUPLICATE KEY UPDATE` clause or `INSERT` statements and returns `NULL` otherwise. Example:

```mysql
INSERT INTO t1 (a,b,c) VALUES (1,2,3),(4,5,6)
  ON DUPLICATE KEY UPDATE c=VALUES(a)+VALUES(b);
```

That statement is identical to the following two statements:  

```mysql
INSERT INTO t1 (a,b,c) VALUES (1,2,3)
  ON DUPLICATE KEY UPDATE c=3;
  
INSERT INTO t1 (a,b,c) VALUES (4,5,6)
  ON DUPLICATE KEY UPDATE c=9;
```



### 别名

Beginning with MySQL 8.0.19, it is possible to use an alias for the row, with, optionally, one or more of its columns to be inserted, following the `VALUES` or `SET` clause, and preceded by the `AS` keyword. Using the row alias  `new`, the statement shown previously using  `VALUES()` to access the new column values can be written in the form shown here: 

```sql
INSERT INTO t1 (a,b,c) VALUES (1,2,3),(4,5,6) AS new
  ON DUPLICATE KEY UPDATE c = new.a+new.b;
```





