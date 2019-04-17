# 20190413

- OCI：oracle调用界面（Oracle数据库的本地接口界面），作用是将从Oracle内核传送而来的查询语句发送至数据库

- sqlplus使用

  ```sql
  PS C:\Users\Administrator\Desktop> sqlplus fmcg_beta/fmcg_beta@172.20.53.150:1521/orcl
  
  SQL*Plus: Release 12.2.0.1.0 Production on Sat Apr 13 10:22:20 2019
  
  Copyright (c) 1982, 2016, Oracle.  All rights reserved.
  
  
  Connected to:
  Oracle Database 11g Enterprise Edition Release 11.2.0.1.0 - 64bit Production
  With the Partitioning, OLAP, Data Mining and Real Application Testing options
  
  /*设置*/
  SQL> help set    --使用help帮助
  SQL> set lines 3000  --设置行宽
  SQL> set pages 1000  --设置分页
  SQL> set timing on  --显示执行时间
  SQL> set null <null>  --数据为null时显示<null>
  
  /*执行SQL*/
  --形式1
  SQL> select empno, deptno from scott.emp where ename = 'SMITH';
  --形式2
  SQL> select empno, deptno from scott.emp where ename = 'SMITH'
    2  ;
  --形式3
  SQL> select empno, deptno from scott.emp where ename = 'SMITH'
    2  /
  --形式4
  SQL> select empno, deptno from scott.emp where ename = 'SMITH'
    2  --再次按下回车，会将sql存储在sqlplus缓冲器中
  SQL> / --执行sql
  --形式5
  SQL> edit test.sql  --在当前路径（C:\Users\Administrator\Desktop）中创建sql脚本，若不存在该文件则新建，若存在则打开文件，这个属于sqlplus指令，末尾不用加;
  SQL> @test.sql  --执行sql脚本，或者使用start test.sql，这个属于sqlplus指令，末尾不用加;
  ```



## 核心SQL语句：SELECT、INSERT、UPDATE、DELETE、MERGE

- **SELECT语句**，数字代表执行先后顺序
      5	SELECT 	<column list>	--select列表
      1	FROM   	<source object list>	--from子句
      1.1	FROM	<left source object> <join type>	--from子句
              			JOIN <right source object> ON <on predicates>
      2	WHERE 	<where predicates>	--where子句
      3	GROUP BY <group by expression(s)>	--group by子句
      4	HAVING	<having predicates>	--having子句
      6	ORDER BY <order by list>	--order by子句

  - from子句

    - source object：数据源对象，包括表、视图、物化视图、分区或子分区、子查询
    - 联结类型
      - 交叉联结：join，笛卡尔乘积
      - 内联结：inner join
      - 外联结：outter join
    - 联结谓词on

  - where子句

    - SQL中的逻辑比较的可能结果是true、false或**未知**

      **当值为null时，逻辑比较的结果就是未知**

    - 空值null代表一个相应值的确实，会影响SQL语句的执行

  - group by子句

    - 将执行from和where子句后得到的经过筛选后的结果集进行**聚合**
    - 并不确定结果数据的排序

  - having子句

    - 将分组汇总后的查询结果限定为只有该子句中的条件为真的数据行

  - select列表

    - 列出查询的返回值最终结果集中需要显示哪些列
    - 这些列可以为**实际列**、**表达式**、**标量子查询结果**（结果仅一行一列）

  - order by子句

    - 对查询最终返回的结果集进行排序
    - 较小的排序会完全在内存中实现，而较大的排序将不得不使用临时磁盘空间来完成

- **INSERT语句**

  insert语句用来向表、分区或视图中添加行

  数据来源：显示列出、通过子查询获取

  - 单表插入

  ```sql
  // 数据显示列出
  insert into hr.jobs(job_id,job_title,min_salary,max_salary) 
  	values('IT_PM','Project Manager',5000,11000);
  // 数据通过子查询获取
  insert into scott.bonus(ename,job,sal)
  	select ename,job,sal * 0.10 from scott.emp;
  ```

  - 多表插入

  ```sql
  insert all
  	when sum_orders < 10000 then 
  into small_customers
  	when sum_orders >= 10000 and sum_orders < 100000 
  into medium_customers
  	else 
  into large_customers
  	select customer_id,sum(order_total) sum_orders from oe.orders 
  		group by custome_id;	
  ```

- **UPDATE语句**

  - 改变表中原有行的列值
    - update子句：指定要更新的表
    - set子句：指定改变列及调整的值
    - where子句：筛选需要更新的行，不约束的话为全部行

  ```sql
  // 形式1：一般形式
  update employees2 
  set salary = salary * 1.10 
  where department_id = 90;
  // 形式2：set子句使用子查询
  update employees
  set salary = (select employees2.salary from employees2 
                where employees2.employee_id = employees.employee_id and 
                	employees.salary != employees2.salary)
  where department_id = 90;
  // 形式3：where子句使用子查询
  update employees2 
  set salary = salary * 1.10 
  where department_id in (select department_id form departments
                         where department_name = 'Executive');
  // 形式4：update子句使用子查询定义了一个表，并对表中的内容进行修改
  update (select e1.salary, e2.salary new_sal from employees e1,employees2 e2
         where e1.employee_id = e2.employee_id and e1.department_id = 90)
  set salary = new_sal;
  ```

- **DELETE语句**

  - 从表中移除数据行
    - delete关键字：可以与hint（提示）关键字结合
    - from子句：指定需要删除数据的表
    - where子句：指定需要删除的行，若不指定则删除全部行

  ```sql
  // 形式1
  delete 
  from employees2
  where department_id = 90;
  // 形式2
  delete
  from (select * from employees2 where department_id = 90);
  // 形式3
  delete
  from employees2 
  where department_id in (select department_id form departments
                         where department_name = 'Executive');
  ```

- **MERGE语句**

  - 按条件获取要更新或插入到表中的数据行，然后从1个或多个源头对表进行更新或者向表中插入行两方面的能力
  - 经常被用在数据仓库中来移动大量的数据
  - 方便地将多个操作结合成一个

  ```sql
  // 语法格式
  merge <hint>
  into <table_name>
  using <table_view_or_query>
  on (<condition>)
  when matched then <update_clause>
  delete <where_clause>
  when not matched then <insert_clause>
  [log errors <log_errors_clause> <reject limit <integer | unlimited>>];
  
  // 实际应用
  merge
  into dept60_bonuser b
  using (select employee_id,salary,department_id from employees 
      where department_id = 60) e
  on (b.employee_id = e.employee_id) --进行匹配的条件
  when matched then --匹配上
  	update set b.bonus_amt = e.salary * 0.2 where b.bonus_amt = 0 --更新数据
  	delete where (e.salary > 7500) --删除数据
  when not matched then --匹配不上
  	insert (b.employee_id,b.bonus_amt) values(e.employee_id,e.salary*0.1) --插入数据
  	where (e.salary < 7500) --插入数据需要满足的条件
  ```

  

## SQL执行

```mer

```



























































# 20190415

## 1、[@MappedSuperclass的用法](https://www.cnblogs.com/zqyanywn/p/7753596.html)

**这个注解表示在父类上面的，用来标识父类。**

基于代码复用和模型分离的思想，在项目开发中使用JPA的@MappedSuperclass注解将实体类的多个属性分别封装到不同的非实体类中。例如，数据库表中都需要id来表示编号，id是这些映射实体类的通用的属性，交给jpa统一生成主键id编号，那么使用一个父类来封装这些通用属性，并用@MappedSuperclas标识。

注意:

1.标注为@MappedSuperclass的类将不是一个完整的实体类，他将不会映射到数据库表，但是他的属性都将映射到其子类的数据库字段中。

2.标注为@MappedSuperclass的类不能再标注@Entity或@Table注解，也无需实现序列化接口。





## 2、[@AttributeOverrides 和 @AttributeOverride](https://blog.csdn.net/sunrainamazing/article/details/80818672)

@AttributeOverrides 和 @AttributeOverride
    用于覆盖多个属性或字段的映射。
	
@AttributeOverride
	String   name   (必填)如果正在使用基于属性的访问，则映射被覆盖的属性的名称，或者如果使用基于字段的访问，则该字段的名称。  

​	Column   column   (必需)被映射到持久属性的列。

- 用于覆盖Basic(无论显式还是默认)属性或字段或Id属性或字段的映射。

- 可以应用于扩展映射超类或嵌入字段或属性的实体，以覆盖映射超类或可嵌入类(或其某个属性的可嵌入类)定义的基本映射或ID映射。


- 可以应用于包含可嵌入类的实例的元素集合或应用于其键和/或值为可嵌入类的映射集合的元素集合。
    当AttributeOverride应用于Map对象时，必须使用 "key." 或"value."作为被覆盖的属性名称的前缀，以便将其指定为 map的key或map的值一部分。

- 要覆盖多级嵌入的映射，必须在name元素中使用点(".")表示形式来表示嵌入属性中的属性。与点符号一起使用的每个标识符的值是相应嵌入字段或属性的名称。
- 如果AttributeOverride未指定，则列映射与原始映射中的相同。