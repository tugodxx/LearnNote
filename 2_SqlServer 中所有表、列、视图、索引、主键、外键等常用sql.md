# SqlServer 中所有表、列、视图、索引、主键、外键等常用sql

## 在数据库内创建的每个对象（约束、默认值、日志、规则、存储过程等）在表中占一行。只有在 tempdb 内，每个临时对象才在该表中占一行。sysobjects 表结构：
| 列名 | 数据类型| 描述          |
| -------- | ------------ | ------------- |
| name     | sysname      | 对象名,常用列 |
| id | int | 对象标识号 |
| xtype | char(2) | 对象类型。常用列。xtype可以是下列对象类型中的一种：<br> C = CHECK 约束<br/>D = 默认值或 DEFAULT 约束<br/>F = FOREIGN KEY 约束<br/>L = 日志<br/>FN = 标量函数 <br/>IF = 内嵌表函数 <br/>P = 存储过程 <br/>PK = PRIMARY KEY 约束（类型是 K） <br/>RF = 复制筛选存储过程<br/>S = 系统表 <br/>TF = 表函数 <br/>TR = 触发器 <br/>U = 用户表 <br/>UQ = UNIQUE 约束（类型是 K） <br/>V = 视图<br>X = 扩展存储过程 |
| uid | smallint | 所有者用户对象编号 |
| info | smallint | 保留。仅限内部使用 |
| status | int | 保留。仅限内部使用 |
| base_schema_ ver | int | 保留。仅限内部使用 |
| replinfo | int | 保留。供复制使用 |
| parent_obj | int | 父对象的对象标识号（例如，对于触发器或约束，该标识号为表 ID）。 |
| crdate | datetime | 对象的创建日期。 |
| ftcatid | smallint | 为全文索引注册的所有用户表的全文目录标识符，对于没有注册的所有用户表则为 0 |
| schema_ver | int | 版本号，该版本号在每次表的架构更改时都增加。 |
| stats_schema_ ver | int | 保留。仅限内部使用。 |
| type | char(2) | 对象类型。可以是下列值之一：<br/> C = CHECK 约束   <br/>D = 默认值或 DEFAULT 约束 <br/>F = FOREIGN KEY 约束<br/> FN = 标量函数 <br>IF = 内嵌表函数  <br>K = PRIMARY KEY 或 UNIQUE 约束<br/> L = 日志<br>P = 存储过程 <br>R = 规则 <br> RF = 复制筛选存储过程<br/>S = 系统表  <br>TF = 表函数 <br>TR = 触发器 <br>U = 用户表 <br>V = 视图 <br>X = 扩展存储过程 |
| userstat | smallint | 保留。 |
| sysstat | smallint | 内部状态信息 |
| indexdel | smallint | 保留 |
| refdate | datetime | 留用 |
| version | int | 保留 |
| deltrig | int | 保留 |
| instrig | int | 保留 |
| updtrig | int | 保留 |
| seltrig | int | 保留 |
| category | int | 用于发布、约束和标识 |
| cache | smallint | 保留 |

[TOC]

## 根据sysobjects 表格我们可以得到如下的查询:

## 1.获取所有表结构

```mssql
select name as [表名] from sysobjects where xtype='U'and name !='dtproperties' 
```

## 2.获取所有的列

```mssql
SELECT
  d.name 表名,
  a.name 字段名,
  (
    CASE
      WHEN Columnproperty(a.id, a.name, 'IsIdentity') = 1 THEN '是'
      ELSE '否'
    END
  ) 标识,
  (
    CASE
      WHEN Columnproperty(a.id, a.name, 'IsIdentity') = 1 THEN IDENT_Seed(d.name)
      ELSE 0
    END
  ) 标识种子,
  (
    CASE
      WHEN Columnproperty(a.id, a.name, 'IsIdentity') = 1 THEN Ident_Incr(d.name)
      ELSE 0
    END
  ) 标识增长量,
  (
    CASE
      WHEN (
        SELECT
          Count(*)
        FROM sysobjects
        WHERE
          (
            name IN (
              SELECT
                name
              FROM sysindexes
              WHERE
                (id = a.id)
                AND (
                  indid IN (
                    SELECT
                      indid
                    FROM sysindexkeys
                    WHERE
                      (id = a.id)
                      AND (
                        colid IN (
                          SELECT
                            colid
                          FROM syscolumns
                          WHERE
                            (id = a.id)
                            AND (name = a.name)
                        )
                      )
                  )
                )
            )
          )
          AND (xtype = 'PK')
      ) > 0 THEN '是'
      ELSE '否'
    END
  ) 主键,
  b.name 类型,
  a.length 占用字节数,
  Columnproperty(a.id, a.name, 'PRECISION') AS 长度,
  Isnull(Columnproperty(a.id, a.name, 'Scale'), 0) AS 小数位数,
  (
    CASE
      WHEN a.isnullable = 1 THEN '是'
      ELSE '否'
    END
  ) 允许空,
  Isnull(e.text, '') 默认值,
  Isnull(g.[value], ' ') AS [说明]
FROM syscolumns a
LEFT JOIN systypes b ON a.xtype = b.xusertype
left JOIN sysobjects d ON a.id = d.id
  AND d.xtype = 'U'
  AND d.name <> 'dtproperties'
LEFT JOIN syscomments e ON a.cdefault = e.id
LEFT JOIN sys.extended_properties g ON a.id = g.major_id
  AND a.colid = g.minor_id
LEFT JOIN sys.extended_properties f ON d.id = f.class
  AND f.minor_id = 0
WHERE
  b.name IS NOT NULL
  and d.name is not null --and  d.name='{0}' --如果只查询指定表,加上此条件
ORDER BY
  a.id,
  a.colorder
```

## 3.获取所有的视图

```mssql
select
  b.name as [视图名称],
  a.text as [视图脚本]
from syscomments a
inner join sysobjects b on a.id = b.id
where
  b.type = 'V'
```

## 4.获取所有主键约束

```mssql
SELECT
  tab.name AS [表名],
  idxCol.is_descending_key as [是否降序],
  idx.name AS [约束名称],
  idx.type_desc as [约束类型],
  col.name AS [约束列名]
FROM sys.indexes idx
JOIN sys.index_columns idxCol ON (
    idx.object_id = idxCol.object_id
    AND idx.index_id = idxCol.index_id
    AND idx.is_primary_key = 1
  )
JOIN sys.tables tab ON (idx.object_id = tab.object_id)
JOIN sys.columns col ON (
    idx.object_id = col.object_id
    AND idxCol.column_id = col.column_id
  );
```

## 5.获取所有唯一约束

```mssql
SELECT
  tab.name AS [表名],
  idxCol.is_descending_key as [是否降序],
  idx.name AS [约束名称],
  idx.type_desc as [约束类型],
  col.name AS [约束列名]
FROM sys.indexes idx
JOIN sys.index_columns idxCol ON (
    idx.object_id = idxCol.object_id
    AND idx.index_id = idxCol.index_id
    AND idx.is_unique_constraint = 1
  )
JOIN sys.tables tab ON (idx.object_id = tab.object_id)
JOIN sys.columns col ON (
    idx.object_id = col.object_id
    AND idxCol.column_id = col.column_id
  );
```

## 6.获取所有外键约束

```mssql
select
  oSub.name AS [子表名称],
  fk.name AS [外键名称],
  SubCol.name AS [子表列名],
  oMain.name AS [主表名称],
  MainCol.name AS [主表列名]
from sys.foreign_keys fk
JOIN sys.all_objects oSub ON (fk.parent_object_id = oSub.object_id)
JOIN sys.all_objects oMain ON (fk.referenced_object_id = oMain.object_id)
JOIN sys.foreign_key_columns fkCols ON (fk.object_id = fkCols.constraint_object_id)
JOIN sys.columns SubCol ON (
    oSub.object_id = SubCol.object_id
    AND fkCols.parent_column_id = SubCol.column_id
  )
JOIN sys.columns MainCol ON (
    oMain.object_id = MainCol.object_id
    AND fkCols.referenced_column_id = MainCol.column_id
  )
```

## 7.获取所有Check约束

```mssql
SELECT
  tab.name AS [表名],
  chk.name AS [约束名称],
  col.name AS [约束列名],
  chk.definition AS [约束定义]
FROM sys.check_constraints chk
JOIN sys.tables tab ON (chk.parent_object_id = tab.object_id)
JOIN sys.columns col ON (
    chk.parent_object_id = col.object_id
    AND chk.parent_column_id = col.column_id
  )
```

## 8.获取所有默认约束

```mssql
SELECT
  tab.name AS [表名],
  def.name AS [约束名称],
  col.name AS [约束列名],
  def.definition AS [约束定义]
FROM sys.default_constraints def
JOIN sys.tables tab ON (def.parent_object_id = tab.object_id)
JOIN sys.columns col ON (
    def.parent_object_id = col.object_id
    AND def.parent_column_id = col.column_id
  )
```



## 9.获取所有索引约束

```mssql
SELECT
  tab.name AS [表名],
  idx.is_unique as [是否唯一索引],
  idxCol.is_descending_key as [是否降序],
  idx.name AS [约束名称],
  idx.type_desc as [约束类型],
  col.name AS [约束列名]
FROM sys.indexes idx
JOIN sys.index_columns idxCol ON (
    idx.object_id = idxCol.object_id
    AND idx.index_id = idxCol.index_id
    AND idx.is_unique_constraint = 0
    and is_primary_key = 0
  )
JOIN sys.tables tab ON (idx.object_id = tab.object_id)
JOIN sys.columns col ON (
    idx.object_id = col.object_id
    AND idxCol.column_id = col.column_id
  );
```

