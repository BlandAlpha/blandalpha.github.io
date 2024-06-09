---
title: SQLite 数据库基础操作 1
author: guoao
category: [笔记]
tags: [计算机, 笔记, 数据库, SQL, SQLite]
image:
   path: https://cdn.pixabay.com/photo/2021/11/03/11/03/chart-6765401_1280.jpg
---

儿童节快乐啊！最近正在忙选修课大作业。是一个QT Quick / C++的软件。也是布鲁伊相关！这里是 [开源地址](https://github.com/BlandAlpha/bluey_gallery) ，有兴趣可以看看哦。

这次的SQLite笔记其实也是项目中的一环。但是正好考研也需要用到数据库，所以就当作复习笔记了。后面也许会把布鲁伊项目的制作过程也写一篇博客。不过要做的事情实在太多了！真希望可以不用睡觉来完成我的项目们啊。

---

> 本文大部分采用 `SQL` 语句，且暂时不涉及*数据库的约束*，仅作最简单的创建与增删改查。
{: .prompt-info}

## 创建SQLite数据库

使用命令行打开 `SQLite3.exe` 所在的文件夹，输入语句：

``` console
$ sqlite3 filename.db
```

此时会在文件夹内创建一个文件名为 `filename.db`的数据库。但此时文件并**不会**保存。需要先查询一下：

```sql
.tables
```

这条语句会显示当前数据库中所有的表，此时才会真正创建文件 `filename.db`，你可以在文件夹内看到这个文件。

如果你没有使用第一条语句而直接进入了 `SQLite` 中，那么请使用下面这条语句创建数据库：

```sql
.open filename.db
```

此时不需要使用 `.tables` 进行创建文件。当然，如果 `filenam.db` 已经存在，则会打开它。

## 创建数据表 CREATE

```sql
CREATE TABLE userinfo (username, password);
```

这条语句会在数据库中创建一个名为 `userinfo` 的表，其中包含两个条目：

- `username`
- `password`

> `SQL` 和 `SQLite` 关键字大小写都不敏感。 `CREATE` 与 `create` 同样有效。但大写有助于区分关键字与表格名称，增加可读性。
{: .prompt-tip}

## 数据表操作

> 破坏性数据表操作可能会导致**不可逆后果**。操作前最好做好备份！
{: .prompt-warning}

### 增加条目 INSERT INTO

```sql
INSERT INTO userinfo (username, password)
VALUES ('user1', 12345);
```

这条语句会在 `userinfo` 表中添加一行条目，其中记录：

- `username` 的值为 `user1`
- `password` 的值为 `12345`

> 语句可以分行以增加可读性。一般来说 `SQL` 会将分号作为语句结束。
{: .prompt-tip}

### 删除条目 DELETE

#### 清空行

```sql
DELETE FROM userinfo
WHERE username = 'user1';
```

这条语句会清空`userinfo` 表中 `user1` 行的所有记录。

#### 清空表

```sql
DELETE FROM userinfo;
```

这条语句会清空整个 `userinfo` 表。

### 更改条目 UPDATE

#### 更改所有行

```sql
UPDATE userinfo
SET password = 54321;
```

这条语句会将 `userinfo` 表中所有的 `password` 更改为 `54321`。

#### 更改指定条目 

```sql
UPDATE userinfo
SET password = 54321
WHERE username = 'user1';
```

该语句会将 `userinfo` 表中 `user1` 的 `password` 更改为 `54321`。

### 查询条目

```sql
SELECT * FROM userinfo;
```

这条语句会查询 `userinfo` 表中的所有数据。

## 数据表修改 ALTER

### 添加列

如果希望在已有的 `userinfo` 表中增加一列 `country` 字段，可以使用以下语法：

```sql
ALTER TABLE userinfo
ADD country;
```

### 删除列

```sql
ALTER TABLE userinfo
DROP id;
```

---

## 数据筛选

假设 `userinfo` 中包含如下条目：

```
| id | username | password | country |
```

我们可以使用一些关键词对数据进行筛选。

### 并且 AND

```sql
SELECT * FROM userinfo
WHERE country = 'CN'
AND id > 25;
```

这条语句会查询 `userinfo` 表中所有 `country` 为 `CN` 且 `id > 50` 的条目。

### 或者 OR

```sql
SELECT * FROM userinfo
WHERE country = 'CN'
OR country = 'USA';
```

这条语句会查询 `userinfo` 表中所有 `country` 为 `CN` 或 `country` 为 `USA` 的条目。

### 结合操作

你可以使用圆括号 `(#)` 组成复杂的表达式：

```sql
SELECT * FROM userinfo
WHERE id > 15
AND (country='CN' OR country='USA');
```

这条语句会查询 `userinfo` 表中所有`id > 15` ，并满足 `country` 为 `CN` 或 `country` 为 `USA` 的条目。

### SELECT DISTINCT

因为不同的用户可能来自相同的国家，因此 `useerinfo` 中 `country` 条目包含许多重复项。例如：

```
username    password    country 
----------  ----------  ----------
admin       12345       CN
user1       23456       US
user2       24442       CN
user3       23244       CN
user4       46361       AU
user5       23244       US
user6       46361       CN
user5       232441445   US
...         ...         ...
```

如果你希望查看用户都来自哪些国家而不产生重复项，可以使用以下语句：

```sql
SELECT DISTINCT country
FROM userinfo;
```

输出类似如下：

```
| country |
+---------+
| CN      |
| US      |
| AU      |
```
