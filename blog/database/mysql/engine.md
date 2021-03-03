# 存储引擎

## 1、MySQL支持哪些存储引擎？

- Innodb
- MyIsam
- memory
- archive

## 2、Innodb和MyIsam的区别？

- Innodb支持事务，myisam不支持事务
- innodb支持外键，myisam不支持外键
- innodb支持mvcc，myisam不支持mvcc
- count(*) 时，myisam会很快，因为他保存了一个变量记录总数，直接获取，innodb需要遍历全表进行统计
- innodb支持表锁、行锁、间隙锁等，myisam只支持表锁
- innodb表必须有主键，即使没有的话，innodb也会以rowid做为主键，myisam可以没主键
- innodb按主键大小有序插入，而myisam按顺序插入

---
收录时间: 2021/01/05

<Vssue :title="$title" />