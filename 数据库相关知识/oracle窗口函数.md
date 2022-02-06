# oracle窗口函数和分析函数

## 1、分析函数的形式

​    分析函数带有一个开窗函数over()，包含三个分析子句:分组(partition by), 排序(order by), 窗口(rows) ，
他们的使用形式如下：分析函数名(参数) over (partition by 子句 order by 子句 rows/range.. 子句)==(注：若窗口函数内和sql语句末尾共存在两个order by，那么sql语句中的排序将最后在分析函数分析结束后执行排序。）==
   a) order by 字段两者一致：即sql语句中的order by子句里的内容和开窗函数over（）中的order by子句里的内容一样，那么sql语句中的排序将先执行,分析函数在分析时就不必再排序；
   b) order by 字段两者不一致：即sql语句中的order by子句里的内容和开窗函数over（）中的order by子句里的内容不一样，

```sql
select e.deptno,
       e.empno,
       e.ename,
       e.sal,
       sum(e.sal)over()  总收入,
       sum(e.sal)over(partition by e.deptno)  部门总收入,--按部门分组求和
       sum(e.sal)over(order by e.empno)  员工累计收入,--按照员工编号（empno）的排序取累计收入和
       sum(e.sal)over(partition by e.deptno order by e.empno)  员工部门内累计收入,--按部门（deptno）分组，同时按员工编号（empno）排序取员工部门内累计收入和
       sum(e.sal)over(partition by e.deptno order by e.empno rows between unbounded preceding and unbounded following)  部门总收入2--可指定范围，结果同上
  from emp e;
```

==注：==

```
--unbounded preceding and unbouned following针对当前所有记录的前一条、后一条记录，也就是表中的所有记录
--unbounded：不受控制的，无限的
--preceding：在...之前
--following：在...之后
rows between unbounded preceding and unbounded following 表中的所有记录
rows between unbounded preceding and current row 是指第一行至当前行的汇总
rows between current row and unbounded following 指当前行到最后一行的汇总
rows between 1 preceding and current row 是指当前行的上一行(rownum-1)到当前行的汇总
rows between 1 preceding and 2 following 是指当前行的上一行(rownum-1)到当前行的下两行(rownum+2)的汇总
```

### 1.1 有关ROWS/RANGE窗口的例子

```sql
with t as
 (select (case
 when level in (1, 2) then
 1
 when level in (4, 5) then
 6
 else
 level
 end) id
 from dual
 connect by level < 10)
 select id,
 sum(id) over(order by id) default_sum,
 sum(id) over(order by id range between unbounded preceding and current row) range_unbound_sum,
 sum(id) over(order by id rows between unbounded preceding and current row) rows_unbound_sum,
 sum(id) over(order by id range between 1 preceding and 2 following) range_sum,
 sum(id) over(order by id rows between 1 preceding and 2 following) rows_sum
from t;
```

  相关的返回结果为：

![image-20210704212211366](E:\study\images\image-20210704212211366.png)

## 2、常用分析函数汇总

### 2.1、row_number() over伪列（添加一个序号

注：此分析函数必须要加order by排序

```sql
select empno,ename,mgr,sal,deptno,row_number() over(order by empno) rn from emp;--按empno排序
select empno,ename,mgr,sal,deptno,row_number() over(partition by deptno order by empno) rn from emp;--（按部门分组，empno排序）添加一个伪列（和rownum类似）
--此分析函数在对数据去重时用的比较多
select * from(
  select empno,ename,mgr,sal,deptno,row_number() over(partition by deptno order by empno) rn from emp)
where rn = 1;--每个部门只取一条数据（当然 partition by 后面可以按需求跟多个字段，来达到你想要的筛选目的）
```

### 2.2、count() over()：计数

```sql
select empno,ename,mgr,sal,deptno,count(*) over() from emp;--总计数
select empno,ename,mgr,sal,deptno,count(*) over(order by empno) from emp;--按照empno累计计数
select empno,ename,mgr,sal,deptno,count(*) over(partition by deptno) from emp;--按照deptno分组计数
select empno,ename,mgr,sal,deptno,count(*) over(partition by deptno order by empno) from emp;--按照deptno分组并累计计数
```

### 2.3、sum() over()：求和

```sql
select empno,ename,mgr,sal,deptno,sum(sal) over() from emp;--总和
select empno,ename,mgr,sal,deptno,sum(sal) over(order by empno) from emp;--按empno累计求和
select empno,ename,mgr,sal,deptno,sum(sal) over(partition by deptno) from emp;--按照deptno分组求和
select empno,ename,mgr,sal,deptno,sum(sal) over(partition by deptno order by empno) from emp;--按照deptno分组并按empno累计求和
```

### 2.4、avg() over()：求平均

```sql
select empno,ename,mgr,sal,deptno,avg(sal) over() from emp;--总平均
select empno,ename,mgr,sal,deptno,avg(sal) over(order by empno) from emp;--按照empno累计平均
select empno,ename,mgr,sal,deptno,avg(sal) over(partition by deptno) from emp;--按照deptno分组求平均
select empno,ename,mgr,sal,deptno,avg(sal) over(partition by deptno order by empno) from emp;--按照deptno分组并按empno一个个累计求平均
```

### 2.5、min() over()；max() over()：求最小最大

```sql
select empno,ename,mgr,sal,deptno
      ,min(sal) over() 最小金额
      ,max(sal) over() 最大金额 from emp;--总最小（大）
select empno,ename,mgr,sal,deptno
      ,min(sal) over(order by empno) 最小金额
      ,max(sal) over(order by empno) 最大金额 from emp;--按empno排序并一个个递增后的最小（大）
select empno,ename,mgr,sal,deptno
      ,min(sal) over(partition by deptno) 最小金额
      ,max(sal) over(partition by deptno) 最大金额 from emp;--按deptno分组后的最小（大）
select empno,ename,mgr,sal,deptno
      ,min(sal) over(partition by deptno order by empno) 最小金额
      ,max(sal) over(partition by deptno order by empno) 最大金额 from emp;--组内、递增累计后的最小（大）
```

### 2.6、rank() over()：跳跃排序；dense_rank()：连续排序；

注：此分析函数同row_number() over()必须要加order by排序

```
select empno,ename,mgr,sal,deptno
      ,rank() over(order by sal desc) 跳跃排序
      ,dense_rank() over(order by sal desc) 连续排序 from emp;--按sal金额排名
select empno,ename,mgr,sal,deptno
      ,rank() over(partition by deptno order by sal desc) 跳跃排序
      ,dense_rank() over(partition by deptno order by sal desc) 连续排序 from emp;--按deptno分组，组内、金额排名
```