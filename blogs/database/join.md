# MySQL 内联、外联、自联
在关系型数据库中，常常会存在多张表，尤其是遵循了数据库的范式设计，表的粒度会拆分的比较细致，
想要获取全面的数据，通常需要同时查询多张表。因此会遵循某种规则将信息并到一块，在 MySQL 中通常有
`INNER JOIN`、`LEFT OUTER JOIN`、`RIGHT OUTER JOIN`、`CROSS JOIN`这几种方式
## 构造表
Person  
+-------------+---------+
| 列名         | 类型     |
+-------------+---------+
| PersonId    | int     |
| FirstName   | varchar |
| LastName    | varchar |
+-------------+---------+  
Address  
+-------------+---------+
| 列名         | 类型    |
+-------------+---------+
| AddressId   | int     |
| PersonId    | int     |
| City        | varchar |
| State       | varchar |
+-------------+---------+  
## INNER JOIN
INNER JOIN 也称之为内联，在联接的过程中，会指定用来匹配的列作条件筛选，通常是主键或外键，只有两张表用来匹配的列有相同
的值才会匹配，最终匹配好的结果是一一对应的，相当于对于匹配列的内容取并集
```mysql
select Person.LastName, Address.City from Person inner join Address on Person.PersonId = Address.PersonId
```
## LEFT OUTER JOIN
左外联，一般也简写为`LEFT JOIN`，和内联不同的是，左表和右表联结的结果是，对于左表没有匹配上右表的行，右表对应的列值赋值为`NULL`，相当于
取左表所有内容和左右表交集内容
```mysql
select Person.LastName, Address.City from Person left join Address using (PersonId)
# using (PersonId) 与 on Person.PersonId = Address.PersonId 等价
```
## RIGHT OUTER JOIN
右外联，一般也简写为`RIGHT JOIN`,它的情况刚好和左外联相反
## CROSS JOIN
左表的每一行和右表的每一行进行联结，相当于笛卡尔积，假设左表有 n 行，右表有 m 行，那最终的结果为 mn 行
## 自联
就是左表和右表都是同一张表，运用内联或外联的方式进行联结


