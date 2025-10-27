 ### 1. **数据模型**

\- **关系型数据库（RDBMS）**  

 \- 采用**表格结构**（行和列）。

 \- 数据之间通过**外键**建立关系。

 \- 遵循**ACID**原则（原子性、一致性、隔离性、持久性）。

 \- 示例：MySQL、PostgreSQL、Oracle、SQL Server。

 

\- **非关系型数据库（NoSQL）**  

 \- 数据模型多样，常见类型包括：

  \- **键值对**（如 Redis、DynamoDB）内存型

  \- **文档型**（如 MongoDB、CouchDB）

  \- **列族型**（如 Cassandra、HBase）

  \- **图数据库**（如 Neo4j）

 \- 不强制要求固定结构，**模式灵活**（schema-less）。

 \- 更注重**可扩展性**和**高性能**，部分系统牺牲强一致性以换取可用性（遵循 CAP 定理中的 AP 或 CP）。

 

\---

 

\### 2. **查询语言**

\- **RDBMS**：使用标准 **SQL**（Structured Query Language）进行数据操作。

\- **NoSQL**：通常使用**特定 API 或查询语法**（如 MongoDB 的 BSON 查询、Cassandra 的 CQL），不一定支持 JOIN 等复杂操作。

 

\---

 

\### 3. **扩展性**

\- **RDBMS**：传统上以**垂直扩展**（增加单机性能）为主，虽然现代方案（如分库分表、NewSQL）也支持水平扩展，但较复杂。

\- **NoSQL**：天然支持**水平扩展**（通过增加节点），适合处理海量数据和高并发场景。

 

\---

 

\### 4. **事务支持**

\- **RDBMS**：强事务支持（完整 ACID）。

\- **NoSQL**：早期 NoSQL 系统弱化事务，但近年来许多（如 MongoDB 4.0+、Cassandra）已支持**多文档/行事务**，不过通常有限制。

 

\---

 

\### 5. **适用场景**

\- **选择关系型数据库**，如果：

 \- 数据结构稳定、关系复杂（如金融、ERP 系统）。

 \- 需要强一致性与完整事务。

\- **选择非关系型数据库**，如果：

 \- 数据结构多变或半结构化（如日志、用户行为数据）。

 \- 需要高吞吐、低延迟、大规模分布式部署（如社交网络、物联网、实时推荐系统）。

 

\---

 

\### 补充：CAP 定理视角

\- RDBMS 通常优先保证 **一致性（C）和可用性（A）**，在网络分区（P）发生时可能牺牲可用性。

\- NoSQL 系统根据设计不同，可能选择 **CP**（如 MongoDB 默认）或 **AP**（如 Cassandra）。

 

好的，根据您提供的资料，我将MySQL的内部结构整理如下：

 

\---

 

\### **MySQL 内部结构概览**

 

MySQL 的整体架构可以清晰地划分为两大核心部分：**服务层 (Server Layer)** 和 **存储引擎层 (Storage Engine Layer)**。这种分层设计使得 MySQL 具有高度的灵活性和可扩展性。

 

\---

![img](file:////Users/jiamaroady/Library/Containers/com.kingsoft.wpsoffice.mac.global/Data/tmp/wps-jiamaroady/ksohtml//wps1.jpg) 

\### **第一部分：服务层 (Server Layer)**

 

服务层是 MySQL 的“大脑”，负责处理客户端请求、解析 SQL、优化执行计划以及管理连接和缓存等核心功能。它独立于底层的数据存储方式，所有跨存储引擎的功能（如存储过程、触发器、视图、内置函数等）都在此层实现。

 

**服务层主要组件及执行流程：**

 

1. **客户端请求**

  \*  客户端发起 SQL 查询或操作请求。

 

2. **连接器 (Connection Handler)**

  \*  **功能**：负责与客户端建立连接、验证用户身份、分配权限。

  \*  **作用**：确保只有合法用户才能访问数据库。

 

3. **查询缓存 (Query Cache)**

  \*  **功能**：检查当前查询是否已存在于缓存中。

\*  **作用**：如果命中缓存，则直接返回结果，无需后续解析和执行，极大提升性能。

*(注：在 MySQL 8.0 中已移除)*

 

4. **分析器 (Parser)**

  \*  **功能**：对 SQL 语句进行词法分析和语法分析。

  \*  **作用**：判断 SQL 语句是否符合语法规则，并识别出表名、字段名等元素。同时会校验所查询的字段是否存在。

 

5. **优化器 (Optimizer)**

  \*  **功能**：为 SQL 语句生成最优的执行计划。

  \*  **作用**：决定使用哪个索引、表的连接顺序等，以最小化资源消耗并提高查询效率。

 

6. **执行器 (Executor)**

  \*  **功能**：调用存储引擎提供的接口来执行最终的读写操作。

  \*  **作用**：在执行前会再次检查用户权限，然后通过存储引擎层获取或修改数据，并将结果返回给客户端。

 

7. **返回结果**

  \*  执行器将从存储引擎层获取的数据返回给客户端。如果启用了查询缓存且命中，则由查询缓存直接返回结果。

 

\---

 

\### **第二部分：存储引擎层 (Storage Engine Layer)**

 

存储引擎层是 MySQL 的“肌肉”，负责实际的数据存储、读取和管理。其核心特点是**插件式架构 (Pluggable Architecture)**，允许用户根据不同的需求选择不同的存储引擎。

 

**主要特点与常用引擎：**

 

\*  **插件式架构**：支持多种存储引擎，如 `InnoDB`、`MyISAM`、`Memory` 等，可以根据应用场景灵活切换。

\*  **InnoDB**：

  \*  **地位**：自 MySQL 5.5.5 版本起成为默认的存储引擎。

  \*  **优势**：支持事务（ACID）、行级锁、外键约束，适合高并发、需要数据一致性的场景。

\*  **其他引擎**：

  \*  **MyISAM**：早期默认引擎，不支持事务和外键，但读取速度快，适用于只读或读多写少的场景。

  \*  **Memory**：数据存储在内存中，速度极快，但重启后数据丢失，适合临时表或缓存。

 

\---

 

\### **总结**

 

MySQL 的两层架构设计使其功能强大且灵活：

\*  **服务层** 提供统一的接口和高级功能，屏蔽了底层存储的差异。

\*  **存储引擎层** 则专注于数据的物理存储和高效访问，通过插件化支持多样化的业务需求。

 

理解这两层及其交互流程，是掌握 MySQL 性能优化和故障排查的基础。

 

`DROP`、`TRUNCATE` 和 `DELETE` 是 MySQL（以及其他关系型数据库）中用于删除数据或对象的三个关键命令，它们在功能上有部分重叠，但本质、作用范围、性能、事务行为等方面存在显著差异。下面从多个维度详细对比它们的**共同点与区别**：

 

\---

 

\## 一、共同点

 

| 方面 | 说明 |

|------|------|

| **目的** | 都用于“删除”数据或数据库对象，最终结果可能是数据消失。 |

| **需要权限** | 执行这些操作都需要相应的数据库权限（如 `DROP` 权限、`DELETE` 权限等）。 |

| **可被审计/日志记录** | 在开启二进制日志（binlog）或审计日志的情况下，这些操作通常会被记录（但细节不同）。 |

| **可能影响性能** | 大量操作时都可能对系统性能造成影响（如锁表、I/O 压力等）。 |

 

\---

 

\## 二、核心区别（详细对比）

 

| 特性 | `DELETE` | `TRUNCATE` | `DROP` |

|------|---------|-----------|--------|

| **SQL 类型** | DML（数据操作语言） | DDL（数据定义语言） | DDL（数据定义语言） |

| **作用对象** | 表中的**数据行**（可带条件） | 整个表的**所有数据** | **整个表结构**（包括数据、索引、约束、触发器等） |

| **是否可带 WHERE 条件** | ✅ 可以（如 `DELETE FROM t WHERE id > 10`） | ❌ 不可以（总是删除全部数据） | ❌ 不适用（删除整个表） |

| **是否删除表结构** | ❌ 仅删数据，表结构保留 | ❌ 仅删数据，表结构保留 | ✅ 表结构也被删除 |

| **事务支持** | ✅ 支持事务（可回滚） | ❌ **不支持事务**（在 InnoDB 中，MySQL 8.0 之前不可回滚；8.0+ 在某些情况下可回滚，但通常视为自动提交） | ❌ **不支持事务**（DDL 操作会隐式提交当前事务） |

| **触发器触发** | ✅ 会触发 `DELETE` 触发器 | ❌ **不会触发**任何触发器 | ❌ 不触发（但表被删，触发器也随之消失） |

| **锁机制** | 行锁（InnoDB）或表锁（MyISAM），取决于存储引擎和条件 | 表级锁（先加排他锁，释放所有页） | 表级元数据锁（MDL） |

| **性能** | 较慢（逐行删除，写日志，触发触发器） | **非常快**（直接释放数据页，重置自增计数器） | 快（删除元数据和文件） |

| **自增 ID（AUTO_INCREMENT）** | 不重置（下次插入继续递增） | **重置为初始值**（通常是 1） | 表被删，自增信息随之消失；重建后从 1 开始 |

| **空间释放** | 不立即释放磁盘空间（InnoDB 标记为可重用，需 `OPTIMIZE TABLE`） | **立即释放磁盘空间**（删除数据页） | **立即释放磁盘空间**（删除 `.ibd` 或 `.MYD` 文件） |

| **日志记录（binlog / redo log）** | 记录每一行的删除（row-based）或整条语句（statement-based） | 通常记录为 DDL 语句（如 `TRUNCATE TABLE t`），不记录每行 | 记录 DDL 语句（如 `DROP TABLE t`） |

| **能否用于视图/临时表** | ✅ 可用于视图（如果可更新）、临时表 | ✅ 可用于临时表 | ✅ 可用于临时表（`DROP TEMPORARY TABLE`） |

| **外键约束影响** | 受外键约束限制（如子表有引用则不能删） | 受外键约束限制（InnoDB 中若被引用，`TRUNCATE` 会失败） | 若被其他表外键引用，`DROP` 会失败（除非使用 `CASCADE`，但 MySQL 不支持 `DROP ... CASCADE`） |

 

\---

 

\## 三、使用场景建议

 

| 命令 | 适用场景 |

|------|--------|

| `DELETE` | 需要**有条件地删除部分数据**；需要**事务回滚**；需要**触发器生效**。 |

| `TRUNCATE` | 需要**快速清空整张表**，且**不需要保留任何数据**；**不需要事务回滚**；希望**重置自增 ID**。 |

| `DROP` | **不再需要该表**，要彻底删除表结构和数据；通常用于**重构表结构**或**清理无用表**。 |

 

\---

 

\## 四、补充说明（MySQL 特定）

 

1. **`TRUNCATE` 在 InnoDB 中的行为**：

  \- MySQL 5.7 及之前：`TRUNCATE` 是不可回滚的 DDL。

  \- MySQL 8.0+：在某些隔离级别和配置下，`TRUNCATE` 可能在事务中执行并回滚（但官方仍建议视为不可回滚操作）。

 

2. **`DROP` 与 `TRUNCATE` 对自增 ID 的影响**：

  \- `TRUNCATE` 会重置 `AUTO_INCREMENT`。

  \- `DELETE` 不会重置，即使删除所有行。

 

3. **权限要求不同**：

  \- `DELETE`：需要 `DELETE` 权限。

  \- `TRUNCATE`：需要 `DROP` 权限（因为内部实现类似重建表）。

  \- `DROP`：需要 `DROP` 权限。

 

\---

 

\## 五、总结口诀

 

\> - **DELETE 删行，可回滚，带条件，慢但灵活**。  

\> - **TRUNCATE 清表，快如风，重自增，不可回滚**。  

\> - **DROP 删表，连根拔，结构无，慎用！**

 

理解这三者的区别，能帮助你在数据库维护、数据清理和性能优化中做出更安全、高效的选择。

 

![img](file:////Users/jiamaroady/Library/Containers/com.kingsoft.wpsoffice.mac.global/Data/tmp/wps-jiamaroady/ksohtml//wps2.jpg) 

![img](file:////Users/jiamaroady/Library/Containers/com.kingsoft.wpsoffice.mac.global/Data/tmp/wps-jiamaroady/ksohtml//wps3.jpg) 

以下是图片中的文字内容：

 

12、文件索引和数据库索引为什么使用B+树？（第9个问题的详细回答）**

 

文件与数据库都是需要较大的存储，都不可能全部存储在内存中，需要存储到磁盘上。

 

而所谓索引，则为了数据的快速定位与查找，那么索引的结构组织要尽量减少查找过程中磁盘I/O的存取次数

因此B+树相比B树更为合适。

数据库系统巧妙利用了局部性原理与磁盘预读原理，将一个节点的大小设为等于一个页，

这样每个节点只需要一次I/O就可以完全载入。

红黑树这种结构，高度明显要深的多，并且由于逻辑上很近的节点(父子)物理上可能很远，无法利用局部性。

（但是红黑树如果是在 内存 进行数据库存储的话， 那么这也不算缺点了，所以有了redis）

 

1. 最重要的是，B+树还有一个最大的好处：方便扫库。

B树必须用中序遍历的方法按序扫库，

B+树直接从叶子结点挨个扫一遍就完了

B+树支持range-query非常方便，而B树不支持，这是数据库选用B+树的最主要原因。

 

2. B+tree的磁盘读写代价更低：

B+tree的非叶子结点 没有指向关键字具体信息 的指针(红色部分)，因此其内部结点相对B 树更小。

如果把所有同一内部结点的关键字存放在同一块盘中，由于没有具体内容，那么盘块所能容纳的关键字数量也越多。

一次性读入内存中的需要查找的关键字也就越多，相对来说IO读写次数也就降低了；

 

3. B+tree的查询效率更加稳定：

内部结点并不是指向文件内容，而只是叶子结点中关键字的索引

所以，任何关键字的查找必须走一条从根结点到叶子结点的路。

所有关键字查询的路径长度相同，导致每一个数据的查询效率相当。

 

B树有可能在中间节点找到数据，稳定性不够。

 

![img](file:////Users/jiamaroady/Library/Containers/com.kingsoft.wpsoffice.mac.global/Data/tmp/wps-jiamaroady/ksohtml//wps4.jpg) 

![img](file:////Users/jiamaroady/Library/Containers/com.kingsoft.wpsoffice.mac.global/Data/tmp/wps-jiamaroady/ksohtml//wps5.jpg) 

![img](file:////Users/jiamaroady/Library/Containers/com.kingsoft.wpsoffice.mac.global/Data/tmp/wps-jiamaroady/ksohtml//wps6.jpg) 

![img](file:////Users/jiamaroady/Library/Containers/com.kingsoft.wpsoffice.mac.global/Data/tmp/wps-jiamaroady/ksohtml//wps7.jpg) 

![img](file:////Users/jiamaroady/Library/Containers/com.kingsoft.wpsoffice.mac.global/Data/tmp/wps-jiamaroady/ksohtml//wps8.jpg)

# 6、数据库引擎InnoDB与MyISAM的区别

## InnoDB

事务、隔离、索引、行级锁、热备份

* 是 MySQL 默认的事务型存储引擎,只有在需要它不支持的特性时,才考虑使用其它存储引擎。
* 实现了四个标准的隔离级别,默认级别是可重复读(REPEATABLE READ)。在可重复读隔离级别下,通过多版本并发控制(MVCC)+ 间隙锁(Next-Key Locking)防止幻影读。
* 主索引是聚簇索引,在索引中保存了数据,从而避免直接读取磁盘,因此对查询性能有很大的提升。
* 内部做了很多优化,包括从磁盘读取数据时采用的可预测性读、能够加快读操作并且自动创建的自适应哈希索引、能够加速插入操作的插入缓冲区等。
* 支持真正的在线热备份。其它存储引擎不支持在线热备份,要获取一致性视图需要停止对所有表的写入,而在读写混合场景中,停止写入可能也意味着停止读取。

## MyISAM

* 设计简单,数据以紧密格式存储。对于只读数据,或者表比较小、可以容忍修复操作,则依然可以使用它。
* 提供了大量的特性,包括压缩表、空间数据索引等。
* 不支持事务。
* 不支持行级锁,只能对整张表加锁,读取时会对需要读到的所有表加共享锁,写入时则对表加排它锁。但在表有读取操作的同时,也可以往表中插入新的记录,这被称为并发插入(CONCURRENT INSERT)。

## 总结

| 特性         | InnoDB                                  | MyISAM                            |
| ------------ | --------------------------------------- | --------------------------------- |
| **事务**     | 支持,可以使用 `Commit`和 `Rollback`语句 | 不支持                            |
| **并发**     | 支持表级锁和行级锁                      | 只支持表级锁                      |
| **外键**     | 支持外键                                | 不支持                            |
| **备份**     | 支持在线热备份                          | 不支持在线热备份                  |
| **崩溃恢复** | 崩溃后损坏概率低,恢复速度快             | 崩溃后发生损坏的概率高,恢复速度慢 |
| **其它特性** | -                                       | 支持压缩表和空间数据索引          |

# 7. 索引

## 7.1 什么时候需要建立数据库索引？

在最频繁使用的、用以缩小查询范围的字段,需要排序的字段上建立索引。

以下两种情况不适合:

1. 对于查询中很少涉及的列或者重复值比较多的列
2. 对于一些特殊的数据类型,不宜建立索引,比如文本字段(text)等

## 7.2 覆盖索引是什么?

如果一个索引包含(或者说覆盖)所有需要查询的字段的值,我们就称之为"覆盖索引"。

我们知道在InnoDB存储引擎中,如果不是主键索引,叶子节点存储的是主键+列值。最终还是要"回表",也就是要通过主键再查找一次,这样就会比较慢。覆盖索引就是把要查询出的列和索引是对应的,不做回表操作!

### 7.2.1 覆盖索引的好处

如果一个索引包含所有需要的查询的字段的值,直接根据索引的查询结果返回数据,而无需读表,能够极大的提高性能。因此,可以定义一个让索引包含的额外的列,即使这个列对于索引而言是无用的。

## 7.3 聚集索引与非聚集索引

### 一句话总结

其实我们有主键的话，已经建立了聚集索引～（INNODB）

CREATE INDEX idx_name ON users(name); 	建立非聚簇索引 如果还需查询INDEX， idx_name意外事件的信息需回表查询！

一个索引包含(或者说覆盖)所有需要查询的字段的值,我们就称之为"覆盖索引"		如上若只查询INDEX， idx_name两个信息那就是覆盖了的～

---

#### ❓1. **索引是新建一个表吗？**

**不是。**索引**不是表**，而是**一种数据结构（通常是 B+ 树）**，用来**快速定位表中的数据行**。你可以把它想象成“书的目录”——目录本身不是正文，但能帮你快速找到正文在哪一页。

- 索引存储在磁盘上，和表数据分开（逻辑上关联）。
- 创建索引不会改变原表数据，但会占用额外存储空间。
- 查询时，MySQL 先查索引，再（可能）回表查数据。

---

#### ❓2. **聚簇索引 vs 非聚簇索引**

| 特性               | 聚簇索引（Clustered Index）         | 非聚簇索引（Non-Clustered Index）          |
| ------------------ | ----------------------------------- | ------------------------------------------ |
| **数据存储方式**   | **表数据就存在索引的叶子节点里**    | 叶子节点只存**主键值（或行指针）**         |
| **一张表有几个？** | **只能有一个**（InnoDB 中就是主键） | 可以有多个                                 |
| **是否需要回表？** | 不需要（数据就在索引里）            | 通常需要（先查索引，再用主键查聚簇索引）   |
| **MySQL 引擎**     | InnoDB 默认使用                     | MyISAM 使用；InnoDB 的二级索引也是非聚簇的 |

> ✅ **InnoDB 的特点**：
>
> - 主键索引 = 聚簇索引 → 叶子节点 = 完整行数据
> - 其他索引（如 `idx_name`）= 非聚簇索引 → 叶子节点 = 主键值

---

#### 🧪 实际例子：一张用户表

```sql
-- 创建用户表（InnoDB 引擎）
CREATE TABLE users (
    id INT PRIMARY KEY,        -- 聚簇索引
    name VARCHAR(50),
    age INT,
    email VARCHAR(100)
);

-- 插入数据
INSERT INTO users VALUES
(1, 'Alice', 25, 'alice@example.com'),
(2, 'Bob', 30, 'bob@example.com'),
(3, 'Charlie', 35, 'charlie@example.com');
```

表中物理存储（InnoDB 聚簇索引结构）大致如下（按主键 `id` 排序）：

| id (PK) | name    | age  | email               |
| ------- | ------- | ---- | ------------------- |
| 1       | Alice   | 25   | alice@example.com   |
| 2       | Bob     | 30   | bob@example.com     |
| 3       | Charlie | 35   | charlie@example.com |

---

#### 🔍 查询示例 1：通过主键查询（聚簇索引）

```sql
SELECT * FROM users WHERE id = 2;
```

✅ **执行过程**：

- 直接在聚簇索引（主键 B+ 树）中查找 `id=2`；
- **叶子节点就是完整行数据** → 直接返回；
- **无需回表**，速度最快。

**结果**：

| id   | name | age  | email           |
| ---- | ---- | ---- | --------------- |
| 2    | Bob  | 30   | bob@example.com |

---

#### 🔍 查询示例 2：通过 name 查询（非聚簇索引）

先创建一个普通索引：

```sql
CREATE INDEX idx_name ON users(name);
```

这个 `idx_name` 是**非聚簇索引**，其 B+ 树结构大致如下：

| name（索引键） | 主键 id（存储在叶子节点） |
| -------------- | ------------------------- |
| Alice          | 1                         |
| Bob            | 2                         |
| Charlie        | 3                         |

现在执行查询：

```sql
SELECT * FROM users WHERE name = 'Bob';
```

✅ **执行过程**：

1. 在 `idx_name` 索引中查找 `name='Bob'` → 找到主键 `id=2`；
2. **回表**：用 `id=2` 去聚簇索引中查找完整行数据；
3. 返回结果。

> 这个“回表”操作会多一次 I/O，比主键查询慢。

**结果**同上。

---

#### 🔍 查询示例 3：覆盖索引（避免回表）

```sql
SELECT id, name FROM users WHERE name = 'Bob';
```

✅ **执行过程**：

- `idx_name` 索引的叶子节点包含 `name` 和 `id`（InnoDB 所有二级索引自动包含主键）；
- 查询所需字段 `id, name` **全部在索引中**；
- **无需回表**，直接返回 → 这就是**覆盖索引**！

---

#### 📌 总结关键点

| 问题               | 答案                                        |
| ------------------ | ------------------------------------------- |
| 索引是新表吗？     | ❌ 不是，是辅助查找的数据结构（如 B+ 树）    |
| 聚簇索引存什么？   | ✅**存完整行数据**（InnoDB 中就是主键索引）  |
| 非聚簇索引存什么？ | ✅**存索引列 + 主键值**，查数据需“回表”      |
| 如何避免回表？     | ✅ 使用**覆盖索引**（SELECT 的列都在索引中） |

---

如果你是 macOS 用户 + 教育工作者，可以用这个例子在课堂上演示：

> “聚簇索引就像字典正文按拼音排好——你要查‘安’，直接翻到那一页就看到字和解释；非聚簇索引就像部首检字表——你先查到‘安’在第 10 页，还得再翻到第 10 页看内容。”

需要我画个结构图或生成 `EXPLAIN` 执行计划吗？

### 7.3.1什么是聚集索引?

聚集索引就是按照拼音查询,非聚集索引就是按照偏旁等来进行查询。

其实,我们的汉语字典的正文本身就是一个聚集索引。

比如,我们要查"安"字,就会很自然地翻开字典的前几页,因为"安"的拼音是"an",而按照拼音排序汉字的字典是以英文字母"a"开头并以"z"结尾的,那么"安"字就自然地排在字典的前部。

如果您翻完了所有以"a"开头的部分仍然找不到这个字,那么就说明您的字典中没有这个字;

同样的,如果查"张"字,那您也会将您的字典翻到最后部分,因为"张"的拼音是"zhang"。也就是说,字典的正文部分本身就是一个目录,您不需要再去查其他目录来找到您需要找的内容。

我们把这种正文内容本身就是一种按照一定规则排列的目录称为"聚集索引"。

### 7.3.2 什么是非聚集索引?

如果您认识某个字,您可以快速地从自动中查到这个字。但您也可能会遇到您不认识的字,不知道它的发音,这时候,您就不能按照刚才的方法找到您要查的字,而需要去根据"偏旁部首"查到您要找的字,然后根据这个字后的页码直接翻到某页来找到您要找的字。

但您结合"部首目录"和"检字表"而查到的字的排序并不是真正的正文的排序方法,比如您查"张"字,我们可以看到在查部首之后的检字表中"张"的页码是672页,检字表中"张"的上面是"驰"字,但页码却是63页,"张"的下面是"弩"字,页面是390页。

很显然,这些字并不是真正的分别位于"张"字的上下方,现在您看到的连续的"驰、张、弩"三字实际上就是他们在非聚集索引中的排序,是字典正文中的字在非聚集索引中的映射。

我们可以通过这种方式来找到您所需要的字,但它需要两个过程,先找到目录中的结果,然后再翻到您所需要的页码。

我们把这种目录纯粹是目录,正文纯粹是正文的排序方式称为"非聚集索引"。

### 7.3.3 聚集索引与非聚集索引的区别是什么?

聚集索引和非聚集索引的区别在于,通过聚集索引可以查到需要查找的数据,而通过非聚集索引可以查到记录对应的主键值,再使用主键的值通过聚集索引查找到需要的数据。

聚集索引和非聚集索引的根本区别是表记录的排列顺序和与索引的排列顺序是否一致。

聚集索引(Innodb)的叶节点就是数据节点,而非聚集索引(MyisAM)的叶节点仍然是索引节点,只不过其包含一个指向对应数据块的指针。

## 7.4 创建索引时需要注意什么?

* **非空字段** : 应该指定列为NOT NULL,除非你想存储NULL。在MySQL中,含有空值的列很难进行查询优化,因为它们使得索引、索引的统计信息以及比较运算更加复杂。你应该用0、一个特殊的值或者一个空串代替空值
* **取值离散大的字段** : (变量各个取值之间的差异程度)的列放到联合索引的前面,可以通过count()函数查看字段的差异值,返回值越大说明字段的唯一值越多字段的离散程度高
* **索引字段越小越好** : 数据库的数据存储以页为单位一页存储的数据越多一次IO操作获取的数据越大效率越高
* 唯一、不为空、经常被查询的字段的字段适合建索引

## 7.5 MySQL中有哪些索引?有什么特点?

* **普通索引** : 仅加速查询
* **唯一索引** : 加速查询 + 列值唯一(可以有null)
* **主键索引** : 加速查询 + 列值唯一(不可以有null) + 表中只有一个
* 
* **组合索引** : 多列值组成一个索引,专门用于组合搜索,其效率大于索引合并
* **全文索引** : 对文本的内容进行分词,进行搜索
* **索引合并** : 使用多个单列索引组合搜索
* 
* **覆盖索引** : select的数据列只用从索引中就能够取得,不必读取数据行,换句话说查询列要被所建的索引覆盖
* **聚簇索引** : 表数据是和主键一起存储的,主键索引的叶结点存储行数据(包含了主键值),二级索引的叶结点存储行的主键值。使用的是B+树作为索引的存储结构,非叶子节点都是索引关键字,但非叶子节点中的关键字中不存储对应记录的具体内容或内容地址。叶子节点上的数据是主键与具体记录(数据内容)

## 7.5.1 例子

当然可以。以下是每种索引类型的示例，包含建表、建索引、插入数据、查询语句与结果。

---

### 1. **普通索引**

```sql
CREATE TABLE t1 (id INT, name VARCHAR(50));
CREATE INDEX idx_name ON t1(name);
INSERT INTO t1 VALUES (1, 'Alice'), (2, 'Bob');
SELECT * FROM t1 WHERE name = 'Alice';
```

**结果：**

| id   | name  |
| ---- | ----- |
| 1    | Alice |

---

### 2. **唯一索引（UNIQUE）**

```sql
CREATE TABLE t2 (id INT, email VARCHAR(100));
CREATE UNIQUE INDEX idx_email ON t2(email);
INSERT INTO t2 VALUES (1, 'alice@example.com'), (2, 'bob@example.com');
SELECT * FROM t2 WHERE email = 'bob@example.com';
```

**结果：**

| id   | email           |
| ---- | --------------- |
| 2    | bob@example.com |

---

### 3. **主键索引**

```sql
CREATE TABLE t3 (id INT PRIMARY KEY, name VARCHAR(50));
INSERT INTO t3 VALUES (101, 'Charlie'), (102, 'Diana');
SELECT * FROM t3 WHERE id = 101;
```

**结果：**

| id   | name    |
| ---- | ------- |
| 101  | Charlie |

---

### 4. **组合索引**

```sql
CREATE TABLE t4 (user_id INT, status VARCHAR(20), amount DECIMAL(10,2));
CREATE INDEX idx_user_status ON t4(user_id, status);
INSERT INTO t4 VALUES (1, 'paid', 99.99), (1, 'pending', 50.00), (2, 'paid', 200.00);
SELECT * FROM t4 WHERE user_id = 1 AND status = 'paid';
```

**结果：**

| user_id | status | amount |
| ------- | ------ | ------ |
| 1       | paid   | 99.99  |

---

### 5. **全文索引（WHERE MATCH(content) AGAINST('database' IN NATURAL LANGUAGE MODE);）**

```sql
CREATE TABLE t5 (id INT, content TEXT, FULLTEXT(content));
INSERT INTO t5 VALUES (1, 'MySQL is a relational database'), (2, 'Redis is a key-value store');
SELECT * FROM t5 WHERE MATCH(content) AGAINST('database' IN NATURAL LANGUAGE MODE);
```

**结果：**

| id   | content                        |
| ---- | ------------------------------ |
| 1    | MySQL is a relational database |

---

### 6. **索引合并**

```sql
CREATE TABLE t6 (a INT, b INT);
CREATE INDEX idx_a ON t6(a);
CREATE INDEX idx_b ON t6(b);
INSERT INTO t6 VALUES (10, 100), (20, 200), (30, 100);
SELECT * FROM t6 WHERE a = 10 OR b = 100;
```

**结果：**

| a    | b    |
| ---- | ---- |
| 10   | 100  |
| 30   | 100  |

---

### 7. **覆盖索引**

```sql
CREATE TABLE t7 (id INT PRIMARY KEY, name VARCHAR(50), age INT);
CREATE INDEX idx_name_age ON t7(name, age);
INSERT INTO t7 VALUES (1, 'Eve', 28), (2, 'Frank', 32);
SELECT name, age FROM t7 WHERE name = 'Eve';
```

**结果：**

| name | age  |
| ---- | ---- |
| Eve  | 28   |

---

### 8. **聚簇索引**（InnoDB 主键即聚簇索引）

```sql
CREATE TABLE t8 (id INT PRIMARY KEY, title VARCHAR(100)) ENGINE=InnoDB;
INSERT INTO t8 VALUES (3, 'Third'), (1, 'First'), (2, 'Second');
SELECT * FROM t8 WHERE id = 2;
```

**结果：**

| id   | title  |
| ---- | ------ |
| 2    | Second |

## 7.6 为什么不对表中的每一列都创建索引?

* 当对表中的数据进行增加、删除和修改的时候,索引也要动态的维护,这样就降低了数据的维护速度
* 索引需要占物理空间,除了数据表占数据空间之外,每一个索引还要占一定的物理空间,如果要建立簇索引,那么需要的空间就会更大
* 创建索引和维护索引要耗费时间,这种时间随着数据量的增加而增加

## 7.7 MySQL索引使用的注意事项

MySQL索引通常是被用于提高WHERE条件的数据行匹配时的搜索速度,在索引的使用过程中,存在一些使用细节和注意事项。

### 7.7.1 不要在列上使用函数和运算

不要在列上使用函数,这将导致索引失效而进行全表扫描。

```sql
-- 错误示例
select * from news where year(publish_time) < 2017
```

为了使用索引,防止执行全表扫描,可以进行改造:

```sql
-- 正确示例
select * from news where publish_time < '2017-01-01'
```

同样,不要在列上进行运算,这也将导致索引失效而进行全表扫描:

```sql
-- 错误示例
select * from news where id / 100 = 1
```

为了使用索引,防止执行全表扫描,可以进行改造:

```sql
-- 正确示例
select * from news where id = 1 * 100
```

### 7.7.2 尽量避免使用否定操作符

应该尽量避免在where子句中使用 `!=` 或 `not in` 或 `<>` 操作符,因为这几个操作符都会导致索引失效而进行全表扫描。

尽量避免使用or来连接条件,应该尽量避免在where子句中使用or来连接条件,因为这会导致索引失效而进行全表扫描。

```sql
-- 不推荐
select * from news where id = 1 or id = 2
```

### 7.7.3 多个单列索引并不是最佳选择

MySQL只能使用一个索引,会从多个索引中选择一个限制最为严格的索引,因此,为多个列创建单列索引,并不能提高MySQL的查询性能。

假设,有两个单列索引,分别为 `news_year_idx(news_year)` 和 `news_month_idx(news_month)`。现在,有一个场景需要针对资讯的年份和月份进行查询,那么,SQL语句可以写成:

```sql
select * from news where news_year = 2017 and news_month = 1
```

事实上,MySQL只能使用一个单列索引。

为了提高性能,可以使用复合索引 `news_year_month_idx(news_year, news_month)` 保证news_year和news_month两个列都被索引覆盖。

以下是具体对比示例，展示“两个单列索引” vs “一个复合索引”的实际效果（含建表、数据、查询与结果）：

---

### 📌 场景：资讯表按年月查询

#### 1. **使用两个单列索引（低效）**

```sql
CREATE TABLE news (
    id INT,
    news_year INT,
    news_month INT,
    title VARCHAR(100)
);

CREATE INDEX news_year_idx ON news(news_year);
CREATE INDEX news_month_idx ON news(news_month);

INSERT INTO news VALUES
(1, 2017, 1, 'News A'),
(2, 2017, 2, 'News B'),
(3, 2018, 1, 'News C');

-- 查询
SELECT * FROM news WHERE news_year = 2017 AND news_month = 1;
```

**结果：**

| id   | news_year | news_month | title  |
| ---- | --------- | ---------- | ------ |
| 1    | 2017      | 1          | News A |

> ✅ 能查出结果，但 **MySQL 通常只选一个索引**（如 `news_year_idx`），再在结果中逐行过滤 `news_month = 1`，**无法同时利用两个索引高效定位**。（回表了）

---

#### 2. **使用复合索引（高效）**

```sql
-- 删除旧索引，创建复合索引
DROP INDEX news_year_idx ON news;
DROP INDEX news_month_idx ON news;
CREATE INDEX news_year_month_idx ON news(news_year, news_month);

-- 同样的数据和查询
SELECT * FROM news WHERE news_year = 2017 AND news_month = 1;
```

**结果：**

| id   | news_year | news_month | title  |
| ---- | --------- | ---------- | ------ |
| 1    | 2017      | 1          | News A |

> ✅ **直接通过复合索引精准定位** `(2017, 1)`，**无需回表过滤**，I/O 和 CPU 开销更低。

---

#### 🔍 关键区别（通过 `EXPLAIN` 可验证）

| 方案         | `EXPLAIN` 的 `key` 字段                | `Extra` 字段                    | 效率         |
| ------------ | -------------------------------------- | ------------------------------- | ------------ |
| 两个单列索引 | `news_year_idx`（或 `news_month_idx`） | `Using where`                   | ❌ 需额外过滤 |
| 复合索引     | `news_year_month_idx`                  | （无额外说明 或 `Using index`） | ✅ 精准命中   |

---

#### 💡 补充：复合索引还支持最左前缀查询

```sql
-- 以下查询也能用到 news_year_month_idx 索引
SELECT * FROM news WHERE news_year = 2017;                     -- ✅ 用索引
SELECT * FROM news WHERE news_year = 2017 ORDER BY news_month; -- ✅ 用索引避免 filesort
```

但：

```sql
SELECT * FROM news WHERE news_month = 1;  -- ❌ 无法使用该复合索引（违反最左前缀）
```

---

这就是复合索引在多条件查询中的优势：**一次索引查找，精准命中组合条件**。

### 7.7.4 复合索引的最左前缀原则（！！！）

复合索引遵守"最左前缀"原则,即在查询条件中使用了复合索引的第一个字段,索引才会被使用。因此,在复合索引中索引列的顺序至关重要。如果不是按照索引的最左列开始查找,则无法使用索引。

假设,有一个场景只需要针对资讯的月份进行查询,那么,SQL语句可以写成:

```sql
select * from news where news_month = 1
```

此时,无法使用 `news_year_month_idx(news_year, news_month)` 索引,因为遵守"最左前缀"原则,在查询条件中没有使用复合索引的第一个字段,索引是不会被使用的。

### 7.7.5 范围查询对多列查询的影响

查询中的某个列有范围查询,则其右边所有列都无法使用索引优化查找。

举个例子,假设有一个场景需要查询本周发布的资讯文章,其中的条件是必须是启用状态,且发布时间在这周内。那么,SQL语句可以写成:

```sql
select * from news where publish_time >= '2017-01-02' and publish_time <= '2017-01-08' and enable = 1
```

这种情况下,因为范围查询对多列查询的影响,将导致 `news_publish_idx(publish_time, enable)` 索引中publish_time右边所有列都无法使用索引优化查找。换句话说,`news_publish_idx(publish_time, enable)` 索引等价于 `news_publish_idx(publish_time)`。

对于这种情况,建议:对于范围查询,务必要注意它带来的副作用,并且尽量少用范围查询,可以通过曲线救国的方式满足业务场景。

例如,上面案例的需求是查询本周发布的资讯文章,因此可以创建一个news_weekth字段用来存储资讯文章的周信息,使得范围查询变成普通的查询,SQL可以改写成:

```sql
select * from news where news_weekth = 1 and enable = 1
```

然而,并不是所有的范围查询都可以进行改造。

对于必须使用范围查询但无法改造的情况,建议:不必试图用SQL来解决所有问题,可以使用其他数据存储技术控制时间轴,例如Redis的SortedSet有序集合保存时间,或者通过缓存方式缓存查询结果从而提高性能。

### 7.7.6 索引不会包含有NULL值的列

只要列中包含有NULL值都将不会被包含在索引中,复合索引中只要有一列含有NULL值,那么这一列对于此复合索引就是无效的。

因此,在数据库设计时,除非有一个很特别的原因使用NULL值,不然尽量不要让字段的默认值为NULL。

### 7.7.7 隐式转换的影响

当查询条件左右两侧类型不匹配的时候会发生隐式转换,隐式转换带来的影响就是可能导致索引失效而进行全表扫描。下面的案例中,date_str是字符串,然而匹配的是整数类型,从而发生隐式转换。

```sql
select * from news where date_str = 201701
```

因此,要谨记隐式转换的危害,时刻注意通过同类型进行比较。

### 7.7.8 like语句的索引失效问题（前导% 索引失效--索引也是最左匹配原则）

like的方式进行查询,在 `like "value%"` 可以使用索引,但是对于 `like "%value%"` 这样的方式,执行全表查询,这在数据量小的表,不存在性能问题,但是对于海量数据,全表扫描是非常可怕的事情。

所以,根据业务需求,考虑使用ElasticSearch或Solr是个不错的方案。

# 8、视图

## 8.1修复“不可重复读"

“不可重复读”（Non-Repeatable Read）是数据库事务并发问题之一，**在可重复读（REPEATABLE READ）隔离级别下被修复**。以下是具体修复方式（以 InnoDB 为例）：

---

### ✅ 修复机制：**多版本并发控制（MVCC）**

#### 场景重现（未修复时）：

- 事务 A 第一次读取某行：`age = 25`
- 事务 B 修改该行并提交：`age = 30`
- 事务 A 再次读取同一行：`age = 30` → **不可重复读**

#### InnoDB 如何修复：

1. **事务开始时创建一致性视图（Read View）**
2. **每次 SELECT 读取的是事务开始时已提交的“快照”**
3. 即使其他事务修改并提交了数据，当前事务仍看到**旧版本数据**

---

### 🧪 示例（InnoDB + REPEATABLE READ）

```sql
-- 会话1（事务A）
SET TRANSACTION ISOLATION LEVEL REPEATABLE READ;
START TRANSACTION;
SELECT age FROM users WHERE id = 1;  -- 返回 25

-- 会话2（事务B）
UPDATE users SET age = 30 WHERE id = 1;
COMMIT;

-- 会话1（事务A 继续）
SELECT age FROM users WHERE id = 1;  -- 仍返回 25（不是30！）
COMMIT;
```

✅ **两次读取结果一致** → **不可重复读被修复**

---

### 🔒 补充：间隙锁（Gap Lock）防止幻读

- InnoDB 在 REPEATABLE READ 下还会用 **Next-Key Lock（行锁 + 间隙锁）**
- 阻止其他事务在范围内插入新行，从而**同时解决幻读问题**

---

### 📌 总结

| 问题       | 修复方式                         | 依赖机制             |
| ---------- | -------------------------------- | -------------------- |
| 不可重复读 | 使用**REPEATABLE READ** 隔离级别 | **MVCC 快照读**      |
| 幻读       | 同上（InnoDB 特有）              | **Next-Key Locking** |

> 💡 注意：标准 SQL 中 REPEATABLE READ **不解决幻读**，但 **InnoDB 通过间隙锁额外解决了**。

## 1. 什么是视图（View）

**视图**（View）是基于一个或多个表（或其他视图）的 **虚拟表**。

它本身不存储数据，而是**保存一条 `SELECT` 查询语句**。

当你查询视图时，MySQL 会动态执行该查询并返回结果。

> ✅ **关键点**：视图是“逻辑表”，不是物理表，数据仍存在于基表中。

---

## 2. 为什么需要视图？

### ✅ 简化复杂查询

将多表连接、聚合、子查询等封装成一个简单名字，用户只需 `SELECT * FROM view_name`。

### ✅ 提高安全性

通过视图只暴露部分字段或行（如隐藏身份证号、薪资），限制用户直接访问基表。

### ✅ 逻辑数据独立性

即使底层表结构变化（如字段重命名），只要视图接口不变，上层应用无需修改。

### ✅ 重用 SQL 逻辑

避免在多个地方重复写相同的复杂查询，提升开发效率和一致性。

---

## 3. MySQL 中创建视图的语法

```sql
CREATE [OR REPLACE] [ALGORITHM = {MERGE | TEMPTABLE | UNDEFINED}]
VIEW view_name [(column_list)]
AS
select_statement
[WITH [CASCADED | LOCAL] CHECK OPTION];
```

- `CREATE VIEW`：创建新视图。
- `OR REPLACE`：如果视图已存在，则替换它。
- `AS select_statement`：定义视图的查询逻辑（必须是有效的 `SELECT`）。
- `WITH CHECK OPTION`：确保通过视图插入/更新的数据仍符合视图的筛选条件（仅对可更新视图有效）。

> 💡 大多数场景只需用最简形式：`CREATE VIEW view_name AS SELECT ...`

---

## 4. 创建视图的实际示例

### 示例 1：基于单表的简单视图

假设有一个员工表 `employees`：

```sql
CREATE TABLE employees (
    id INT PRIMARY KEY,
    name VARCHAR(50),
    department VARCHAR(50),
    salary DECIMAL(10,2)
);
```

创建一个只显示“技术部”员工的视图：

```sql
CREATE VIEW tech_employees AS
SELECT id, name, salary
FROM employees
WHERE department = '技术部';
```

使用：

```sql
SELECT * FROM tech_employees;
```

### 示例 2：基于多表连接的复杂视图

假设有订单表 `orders` 和客户表 `customers`，创建一个“客户订单摘要”视图：

```sql
CREATE VIEW customer_order_summary AS
SELECT 
    c.name AS customer_name,
    COUNT(o.id) AS order_count,
    SUM(o.amount) AS total_spent
FROM customers c
JOIN orders o ON c.id = o.customer_id
GROUP BY c.id, c.name;
```

使用：

```sql
SELECT * FROM customer_order_summary WHERE total_spent > 1000;
```

---

## 5. 视图的管理与使用

### 查看视图定义

```sql
SHOW CREATE VIEW view_name;
-- 或
SELECT * FROM INFORMATION_SCHEMA.VIEWS 
WHERE TABLE_NAME = 'view_name';
```

### 修改视图

```sql
-- 方法1：替换
CREATE OR REPLACE VIEW view_name AS ...;

-- 方法2：显式修改（MySQL 也支持）
ALTER VIEW view_name AS ...;
```

### 删除视图

```sql
DROP VIEW [IF EXISTS] view_name;
```

### 查询视图

和查普通表一样：

```sql
SELECT * FROM view_name WHERE ...;
```

> ⚠️ 注意：视图不能直接 `INSERT`/`UPDATE`/`DELETE`，除非满足“可更新视图”的条件（如无聚合、无 DISTINCT、无 GROUP BY 等）。

---

## 6. 注意事项与限制

- **性能**：每次查询视图都会执行底层 `SELECT`，复杂视图可能变慢。必要时可考虑物化中间表。
- **不可索引**：MySQL 不支持对视图创建索引（但可对基表建索引以优化视图查询）。
- **可更新性限制**：包含 `JOIN`、`GROUP BY`、`DISTINCT`、子查询等的视图通常不可更新。
- **权限**：用户需有基表的 `SELECT` 权限才能创建/使用视图。

---

## 7. 小结

| 优势             | 适用场景                  |
| ---------------- | ------------------------- |
| 简化查询逻辑     | 复杂报表、多表关联        |
| 数据安全隔离     | 对外提供受限数据接口      |
| 解耦应用与表结构 | 表结构演进时保持 API 稳定 |

> 🎯 **建议**：在需要重复使用复杂查询、或需要对不同角色提供不同数据视图时，优先考虑使用视图。

---

✅ 视图是数据库设计中非常实用的工具，合理使用能显著提升系统可维护性与安全性。


# 8*游标


## 什么是游标（Cursor）？

游标（**Cursor**）是数据库系统提供的一种 **逐行处理查询结果集** 的机制。

它像一个“指针”，让你可以在 SQL 查询返回的多行数据中，**一行一行地读取、处理甚至修改**，而不是一次性拿到所有结果。

> 💡 简单类比：
> 普通 `SELECT` 是把一整本书的内容打印出来；
> 游标则是一页一页翻着看，还能在每页上做笔记或修改。

---

## 1. 为什么需要游标？

虽然 SQL 本身是**面向集合**（set-based）的语言（擅长批量操作），

但在某些场景下，**必须逐行处理**数据，例如：

- 对每一行执行复杂的业务逻辑（如调用外部 API、条件判断后更新不同字段）
- 无法用单条 SQL 实现的循环计算
- 数据迁移或清洗中需要“一行一规则”的处理

> ⚠️ 注意：**能用集合操作就别用游标**！游标通常性能较差，应作为最后手段。

---

## 2. 游标的基本工作流程（以 MySQL 为例）

在存储过程或函数中使用游标，一般分四步：

1. **声明游标**（DECLARE）
2. **打开游标**（OPEN）
3. **逐行读取**（FETCH）
4. **关闭游标**（CLOSE）

---

## 3. 实际例子：MySQL 中使用游标

假设有一个订单表 `orders`，你想给每个订单金额小于 100 的客户发优惠券（这里用打印代替发券）：

```sql
DELIMITER $$

CREATE PROCEDURE GiveCouponToSmallOrders()
BEGIN
    -- 声明变量
    DECLARE done INT DEFAULT FALSE;
    DECLARE order_id INT;
    DECLARE customer_name VARCHAR(100);

    -- 声明游标：查询小订单的客户
    DECLARE small_order_cursor CURSOR FOR
        SELECT o.id, c.name
        FROM orders o
        JOIN customers c ON o.customer_id = c.id
        WHERE o.amount < 100;

    -- 声明 continue handler：当 FETCH 到末尾时设 done = TRUE
    DECLARE CONTINUE HANDLER FOR NOT FOUND SET done = TRUE;

    -- 打开游标
    OPEN small_order_cursor;

    -- 开始循环
    read_loop: LOOP
        FETCH small_order_cursor INTO order_id, customer_name;
        IF done THEN
            LEAVE read_loop;
        END IF;

        -- 模拟发优惠券（实际可能是 INSERT 或调用其他逻辑）
        SELECT CONCAT('Sending coupon to ', customer_name, ' for order ', order_id) AS message;
    END LOOP;

    -- 关闭游标
    CLOSE small_order_cursor;
END$$

DELIMITER ;
```

调用：

```sql
CALL GiveCouponToSmallOrders();
```

---

## 4. 游标的类型（不同数据库支持不同）

| 类型           | 说明                                                        |
| -------------- | ----------------------------------------------------------- |
| **只读游标**   | 仅能读取数据（MySQL 默认）                                  |
| **可滚动游标** | 可向前/向后移动（MySQL 不支持，但 SQL Server、Oracle 支持） |
| **可更新游标** | 允许通过游标修改基表数据（MySQL 有限支持）                  |

> 🍏 **macOS 用户提示**：在 MySQL Workbench 或命令行中调试游标时，建议先在小数据集上测试，避免卡死。

---

## 5. 游标的缺点与替代方案

### ❌ 缺点：

- **性能差**：逐行处理远慢于集合操作
- **资源占用高**：长时间持有结果集可能锁表
- **代码复杂**：比普通 SQL 更难维护

### ✅ 替代方案（优先考虑）：

- 使用 `JOIN` + `UPDATE` 批量更新
- 用窗口函数（如 `ROW_NUMBER()`）替代循环逻辑
- **在应用层（如 Python、Java）处理循环，而非数据库层**

---

## 6. 小结

| 场景                                  | 是否推荐用游标 |
| ------------------------------------- | -------------- |
| 批量更新、简单过滤                    | ❌ 不推荐       |
| 复杂逐行业务逻辑（且无法用 SQL 表达） | ✅ 可谨慎使用   |
| 高并发、大数据量                      | ❌ 避免使用     |

> 🎯 **记住**：**游标是“瑞士军刀”里的小镊子——有用，但不是锤子**。
> 优先用 SQL 的集合思维解决问题，实在不行再用游标。

---

如果你对工业界中多线程与锁编程感兴趣，其实游标在数据库内部也涉及**行级锁**和**事务隔离**问题——比如在可重复读（RR）隔离级别下，游标遍历过程中其他事务的修改是否可见，这和锁机制密切相关。需要的话我可以进一步展开 😊

# 9、锁

## 一、锁的分类体系

MySQL的锁机制按照粒度可以分为三个级别：

* **全局锁** :锁定整个数据库实例
* **表级锁** :锁定整张表
* **行级锁** :锁定具体的行记录

---

## 二、全局锁(Global Lock)

### 作用

使整个数据库处于只读状态,所有的增删改操作都会被阻塞。

### 使用场景

主要用于 **全库逻辑备份** ,确保备份过程中数据的一致性,避免因数据或表结构的更新导致备份文件数据不一致。

### 缺陷

* 备份大型数据库耗时长
* 备份期间业务只能读取数据,无法更新,造成业务停滞

### 改进方案

在**可重复读隔离级别**下:

* 备份前先开启事务,创建 Read View
* 利用 MVCC 机制,备份期间业务仍可正常更新数据
* 备份看到的始终是事务开启时的数据快照,不受其他事务影响


### 实际操作示例

```sql

-- 加全局锁

FLUSH TABLES WITH READ LOCK;


-- 此时整个数据库只读

-- 在另一个会话中尝试写入会被阻塞

INSERT INTO users (name) VALUES ('张三');  -- 会一直等待


-- 执行备份

mysqldump -u root -p database_name > backup.sql


-- 释放全局锁

UNLOCK TABLES;

```

### 改进方案示例(使用MVCC)

```sql

-- 使用事务进行一致性备份(InnoDB)

START TRANSACTION WITH CONSISTENT SNAPSHOT;


-- 此时可以正常备份,其他事务仍可写入

mysqldump --single-transaction -u root -p database_name > backup.sql


-- 备份完成后提交事务

COMMIT;

```

---

---

## 三、表级锁(Table Lock)

### 表锁特点(MyISAM)

**基本特性:**

* 不会出现死锁
* 锁冲突几率高,并发性能低
* MyISAM 引擎会自动加表锁

**锁模式:**

* **表共享读锁(Table Read Lock)** :允许并发读,阻塞写
* **表独占写锁(Table Write Lock)** :阻塞其他所有读写操作

**加锁规则:**

* 执行 SELECT 前自动加读锁
* 执行增删改前自动加写锁

**阻塞关系:**

* 读锁会阻塞写操作,但不阻塞其他读操作
* 写锁会阻塞所有的读写操作

**局限性:**
MyISAM 不适合写密集型应用,因为写锁会阻塞所有其他操作,导致查询长时间等待甚至永久阻塞。

#### MyISAM 表锁示例

准备测试表：

```sql

CREATE TABLE books (

    id INT PRIMARY KEY,

    title VARCHAR(100),

    stock INT

) ENGINE=MyISAM;


INSERT INTO books VALUES (1, 'MySQL从入门到精通', 100);

```

**读锁示例：**

```sql

-- 会话1: 加读锁

LOCK TABLES books READ;


-- 会话1: 可以读取

SELECT * FROM books;  -- ✓ 成功


-- 会话1: 不能写入

UPDATE books SET stock = 90 WHERE id = 1;  -- ✗ 报错


-- 会话2: 可以读取

SELECT * FROM books;  -- ✓ 成功


-- 会话2: 写入会被阻塞

UPDATE books SET stock = 90 WHERE id = 1;  -- 等待中...


-- 会话1: 释放锁

UNLOCK TABLES;


-- 此时会话2的UPDATE立即执行

```

**写锁示例：**

```sql

-- 会话1: 加写锁

LOCK TABLES books WRITE;


-- 会话1: 可以读写

SELECT * FROM books;  -- ✓ 成功

UPDATE books SET stock = 80 WHERE id = 1;  -- ✓ 成功


-- 会话2: 读写都被阻塞

SELECT * FROM books;  -- 等待中...

UPDATE books SET stock = 70 WHERE id = 1;  -- 等待中...


-- 会话1: 释放锁

UNLOCK TABLES;


-- 此时会话2的操作立即执行

```

---

---

### 1. 元数据锁(Metadata Lock, MDL)

**作用:**
对表进行 CRUD 操作时自动加上,防止其他线程对表结构进行变更。

create read update delete

**释放时机:**
事务提交后才会释放(注意不是语句执行完就释放)。


#### 元数据锁(MDL)示例

```sql

-- 会话1: 开启事务并查询

START TRANSACTION;

SELECT * FROM users WHERE id = 1;

-- 此时自动加上MDL读锁,事务未提交


-- 会话2: 尝试修改表结构

ALTER TABLE users ADD COLUMN age INT;  -- 会被阻塞,等待MDL写锁


-- 会话1: 提交事务

COMMIT;  -- MDL读锁释放


-- 此时会话2的ALTER TABLE立即执行

```

**实际问题场景：**

```sql

-- 会话1: 长事务

START TRANSACTION;

SELECT * FROM orders WHERE user_id = 100;

-- ... 业务处理很久,忘记提交 ...


-- 会话2: DBA想加个索引

ALTER TABLE orders ADD INDEX idx_user_id(user_id);  -- 一直等待


-- 会话3,4,5...: 后续所有涉及orders表的查询都会排队等待

SELECT * FROM orders WHERE id = 1;  -- 都在等待

```

---

### 2. 意向锁(Intention Lock)

**作用:**
表级锁,用于快速判断表中是否有记录被加锁。

**类型:**

* **意向共享锁(IS)** :对某些记录加共享锁前,先在表级加 IS 锁
* **意向独占锁(IX)** :对某些记录加独占锁前,先在表级加 IX 锁

**兼容性:**

* 意向锁之间不冲突
* 意向锁与行级锁不冲突
* 意向锁只与表级的共享锁和独占锁冲突

**注意:**

* 普通 SELECT 不加行级锁,利用 MVCC 实现一致性读
* 可以通过 `SELECT ... LOCK IN SHARE MODE` 或 `SELECT ... FOR UPDATE` 显式加锁

#### 意向锁示例

```sql

-- 准备测试数据

CREATE TABLE accounts (

    id INT PRIMARY KEY,

    name VARCHAR(50),

    balance DECIMAL(10,2)

) ENGINE=InnoDB;


INSERT INTO accounts VALUES (1, '张三', 1000.00), (2, '李四', 2000.00);

```

**意向锁的作用演示：**

```sql

-- 会话1: 对某行加排他锁

START TRANSACTION;

SELECT * FROM accounts WHERE id = 1 FOR UPDATE;

-- InnoDB自动在表级加意向排他锁(IX),在行级加排他锁(X)


-- 会话2: 尝试加表级共享锁

LOCK TABLES accounts READ;  -- 立即失败,因为检测到意向排他锁


-- 如果没有意向锁,会话2需要扫描整个表的每一行来判断是否有行锁

-- 有了意向锁,直接在表级就能快速判断

```

**显式加锁示例：**

```sql

-- 共享锁(读锁)

SELECT * FROM accounts WHERE id = 1 LOCK IN SHARE MODE;


-- 排他锁(写锁)

SELECT * FROM accounts WHERE id = 1 FOR UPDATE;

```


---

### 3. AUTO-INC 锁

**作用:**
保证自增主键值的连续递增。

**传统实现:**
插入数据时加表级 AUTO-INC 锁,为自增字段赋值后,等整个插入语句执行完成才释放锁。

**缺陷:**
大量数据插入时,其他事务的插入操作会被阻塞,影响并发性能。

**轻量级锁改进:**
为自增字段赋值后立即释放锁,无需等待整个语句执行完成。

**控制参数(innodb_autoinc_lock_mode):**

| 值   | 行为               | 说明                                        |
| ---- | ------------------ | ------------------------------------------- |
| 0    | 采用 AUTO-INC 锁   | 语句执行结束后释放                          |
| 1    | 混合模式           | 普通 INSERT 立即释放;批量插入语句结束后释放 |
| 2    | 交错模式(轻量级锁) | 申请自增值后立即释放,性能最高               |

**最佳实践:**

* 设置 `innodb_autoinc_lock_mode = 2`(性能最高)
* 同时设置 `binlog_format = row`(避免主从复制数据不一致)
* 这样既能提升并发性,又能保证数据一致性



#### AUTO-INC 锁示例

```sql

CREATE TABLE orders (

    id INT PRIMARY KEY AUTO_INCREMENT,

    user_id INT,

    amount DECIMAL(10,2)

) ENGINE=InnoDB;

```

**传统AUTO-INC锁(模式0):**

```sql

-- 会话1: 插入数据

INSERT INTO orders (user_id, amount) VALUES (1, 100.00);

-- 获取AUTO-INC表锁 -> 分配id=1 -> 执行插入 -> 释放表锁


-- 会话2: 同时插入

INSERT INTO orders (user_id, amount) VALUES (2, 200.00);

-- 等待会话1的AUTO-INC锁释放...

```

**轻量级锁(模式2):**

```sql

-- 设置为轻量级锁模式

SET GLOBAL innodb_autoinc_lock_mode = 2;


-- 会话1: 插入数据

INSERT INTO orders (user_id, amount) VALUES (1, 100.00);

-- 获取轻量级锁 -> 分配id=1 -> 立即释放锁 -> 执行插入


-- 会话2: 可以立即获取锁分配id

INSERT INTO orders (user_id, amount) VALUES (2, 200.00);

-- 获取轻量级锁 -> 分配id=2 -> 立即释放锁 -> 执行插入


-- 两个插入可以并发执行!

```

**批量插入的区别：**

```sql

-- 模式1(混合模式)

-- 普通INSERT: 立即释放

INSERT INTO orders (user_id, amount) VALUES (1, 100.00);  -- 快速


-- 批量INSERT: 语句结束才释放

INSERT INTO orders (user_id, amount) 

SELECT user_id, amount FROM temp_orders;  -- 其他插入会等待整个语句

```


---

## 四、行级锁(Row Lock)

### 行锁特点(InnoDB)

**基本特性:**

* 可能出现死锁（死锁的形成条件？预防/避免/解开？）
* 锁冲突几率低,并发性能高
* 通过索引实现(与 Oracle 不同)

**关键注意事项:**

1. **必须有索引** :SQL 未走索引会导致全表扫描,行锁升级为表锁
2. **共享锁兼容** :两个事务可以同时持有同一索引的共享锁
3. **独占锁互斥** :独占锁之间不兼容
4. **自动加锁** :INSERT、DELETE、UPDATE 在事务中自动加排他锁

**测试环境准备**

```sql

CREATE TABLE students (

    id INT PRIMARY KEY,

    name VARCHAR(50),

    age INT,

    class_id INT,

    INDEX idx_age(age),

    INDEX idx_class(class_id)

) ENGINE=InnoDB;


INSERT INTO students VALUES 

(1, '张三', 18, 1),

(5, '李四', 20, 1),

(10, '王五', 22, 2),

(15, '赵六', 24, 2);


-- 设置为可重复读隔离级别

SET SESSION TRANSACTION ISOLATION LEVEL REPEATABLE READ;

```


---

### 行锁类型

#### 1. 记录锁(Record Lock)

**作用:**
锁定一条具体的记录。

**类型:**

* 排他锁(X 锁)
* 共享锁(S 锁)

**排他锁（X锁）：我锁住，别人既不能读也不能写；共享锁（S锁）：我锁住，别人还能读，但不能写。**

##### 1. 记录锁(Record Lock)示例

```sql

-- 会话1: 锁定id=5的记录

START TRANSACTION;

SELECT * FROM students WHERE id = 5 FOR UPDATE;

-- 只锁定id=5这一条记录


-- 会话2: 访问其他记录

START TRANSACTION;

SELECT * FROM students WHERE id = 1 FOR UPDATE;  -- ✓ 成功,不冲突

SELECT * FROM students WHERE id = 10 FOR UPDATE;  -- ✓ 成功,不冲突


-- 会话2: 尝试访问id=5

SELECT * FROM students WHERE id = 5 FOR UPDATE;  -- ✗ 阻塞,等待会话1


-- 会话1: 提交

COMMIT;


-- 会话2: 立即获得锁

```


---

#### 2. 间隙锁(Gap Lock)

**作用:**
锁定索引记录之间的间隙,防止其他事务插入数据。

**特点:**

* 只存在于**可重复读隔离级别**
* 用于解决**幻读**问题
* 间隙锁之间是兼容的(不互斥)
* 两个事务可以同时持有包含相同间隙范围的间隙锁



##### 间隙锁(Gap Lock)示例

**场景：防止幻读**

```sql

-- 会话1: 查询age=19的学生(不存在)

START TRANSACTION;

SELECT * FROM students WHERE age = 19 FOR UPDATE;

-- 锁定间隙 (18, 20),防止插入age=19的记录


-- 会话2: 尝试插入age=19的记录

START TRANSACTION;

INSERT INTO students VALUES (3, '新同学', 19, 1);  -- ✗ 阻塞!


-- 会话2: 插入其他年龄

INSERT INTO students VALUES (3, '新同学', 17, 1);  -- ✓ 成功,不在间隙内

INSERT INTO students VALUES (6, '另一个', 21, 1);  -- ✓ 成功,不在间隙内


-- 会话1: 再次查询

SELECT * FROM students WHERE age = 19 FOR UPDATE;

-- 仍然是空结果,没有幻读!


COMMIT;


-- 会话2: 现在可以插入了

```

**间隙锁兼容性示例：**

```sql

-- 会话1: 锁定间隙(18, 20)

START TRANSACTION;

SELECT * FROM students WHERE age = 19 FOR UPDATE;


-- 会话2: 也可以锁定相同的间隙

START TRANSACTION;

SELECT * FROM students WHERE age = 19 LOCK IN SHARE MODE;  -- ✓ 成功!


-- 两个间隙锁不冲突,但都会阻止插入

INSERT INTO students VALUES (3, '测试', 19, 1);  -- ✗ 两个会话都会阻塞这个插入

```


---

#### 3. 临键锁(Next-Key Lock)

**定义:**
Next-Key Lock = Record Lock + Gap Lock

**作用:**
锁定记录本身及记录前面的间隙,既保护记录,又防止间隙中插入新记录。


##### Next-Key Lock(临键锁)示例

```sql

-- 会话1: 范围查询

START TRANSACTION;

SELECT * FROM students WHERE age >= 20 AND age < 24 FOR UPDATE;

-- 锁定: [20, 22] 的记录 + (18, 20) 和 (22, 24) 的间隙

-- 即: (18, 24) 的Next-Key Lock


-- 会话2: 尝试各种操作

START TRANSACTION;


-- 修改锁定范围内的记录

UPDATE students SET name = '李四改' WHERE id = 5;  -- ✗ 阻塞


-- 在锁定范围内插入

INSERT INTO students VALUES (7, '新生', 21, 1);  -- ✗ 阻塞


-- 在锁定范围外操作

UPDATE students SET name = '张三改' WHERE id = 1;  -- ✓ 成功

INSERT INTO students VALUES (17, '新生', 25, 2);  -- ✓ 成功

```


---

#### 4. 插入意向锁(Insert Intention Lock)

**作用:**
事务在插入记录前,需要判断插入位置是否被其他事务的间隙锁(或临键锁)锁定。

**行为:**

* 如果插入位置已被加间隙锁,插入操作会阻塞
* 此时生成插入意向锁,表示有事务想在该区间插入记录,但处于等待状态
* 直到持有间隙锁的事务提交后才能插入

---

## 五、MySQL 加锁规则

### 唯一索引等值查询

| 记录是否存在 | 加锁类型 | 说明                       |
| ------------ | -------- | -------------------------- |
| 存在         | 记录锁   | Next-Key Lock 退化为记录锁 |
| 不存在       | 间隙锁   | Next-Key Lock 退化为间隙锁 |



#### 唯一索引加锁规则示例

**记录存在 - 退化为记录锁：**

```sql

-- 会话1: 等值查询存在的记录

START TRANSACTION;

SELECT * FROM students WHERE id = 5 FOR UPDATE;

-- 只锁定id=5这条记录,不锁间隙


-- 会话2: 可以在间隙中插入

START TRANSACTION;

INSERT INTO students VALUES (3, '新学生', 19, 1);  -- ✓ 成功!

INSERT INTO students VALUES (7, '另一个', 21, 1);  -- ✓ 成功!


-- 但不能修改id=5

UPDATE students SET name = '李四改' WHERE id = 5;  -- ✗ 阻塞

```

**记录不存在 - 退化为间隙锁：**

```sql

-- 会话1: 查询不存在的记录

START TRANSACTION;

SELECT * FROMstudentsWHERE id = 7 FOR UPDATE;

-- 锁定间隙(5, 10),不锁任何记录


-- 会话2: 可以修改已存在的记录

START TRANSACTION;

UPDATE students SET name = '李四改' WHERE id = 5;  -- ✓ 成功!

UPDATE students SET name = '王五改' WHERE id = 10;  -- ✓ 成功!


-- 但不能在间隙内插入

INSERT INTO students VALUES (7, '新学生', 21, 1);  -- ✗ 阻塞

INSERT INTO students VALUES (8, '另一个', 22, 1);  -- ✗ 阻塞

```


---

### 非唯一索引等值查询

| 记录是否存在 | 加锁类型               | 说明                       |
| ------------ | ---------------------- | -------------------------- |
| 存在         | Next-Key Lock + 间隙锁 | 额外加间隙锁               |
| 不存在       | 间隙锁                 | Next-Key Lock 退化为间隙锁 |



#### 非唯一索引加锁规则示例

**记录存在 - Next-Key Lock + 额外间隙锁：**

```sql

-- 会话1: 用非唯一索引查询存在的记录

START TRANSACTION;

SELECT * FROM students WHERE age = 20 FOR UPDATE;

-- 锁定: (18, 20] 的Next-Key Lock + (20, 22) 的间隙锁


-- 会话2: 测试各种操作

START TRANSACTION;


INSERT INTO students VALUES (3, '测试1', 19, 1);  -- ✗ 阻塞,在(18,20)间隙内

INSERT INTO students VALUES (6, '测试2', 20, 1);  -- ✗ 阻塞,age=20被锁

INSERT INTO students VALUES (7, '测试3', 21, 1);  -- ✗ 阻塞,在(20,22)间隙内

INSERT INTO students VALUES (2, '测试4', 17, 1);  -- ✓ 成功,不在锁定范围

```

**记录不存在 - 退化为间隙锁：**

```sql

-- 会话1: 查询不存在的age

START TRANSACTION;

SELECT * FROM students WHERE age = 21 FOR UPDATE;

-- 锁定间隙(20, 22)


-- 会话2: 测试

START TRANSACTION;

UPDATE students SET name = '李四改' WHERE age = 20;  -- ✓ 成功,记录未锁

INSERT INTO students VALUES (7, '测试', 21, 1);  -- ✗ 阻塞,在间隙内

```


---

### 非唯一索引范围查询

**加锁类型:**
Next-Key Lock 不会退化,保持完整的临键锁。


#### 范围查询加锁示例

```sql

-- 会话1: 非唯一索引范围查询

START TRANSACTION;

SELECT * FROM students WHERE age BETWEEN 20 AND 22 FOR UPDATE;

-- Next-Key Lock不退化: (18, 24] 都被锁定


-- 会话2: 测试边界

START TRANSACTION;

INSERT INTO students VALUES (3, '测试1', 19, 1);  -- ✗ 阻塞

INSERT INTO students VALUES (7, '测试2', 21, 1);  -- ✗ 阻塞

INSERT INTO students VALUES (12, '测试3', 23, 1);  -- ✗ 阻塞

INSERT INTO students VALUES (17, '测试4', 25, 2);  -- ✓ 成功

```


### 没有索引导致的表锁示例

```sql

-- 会话1: 查询没有索引的列

START TRANSACTION;

SELECT * FROM students WHERE name = '张三' FOR UPDATE;

-- 因为name没有索引,全表扫描,锁定整个表!


-- 会话2: 所有操作都被阻塞

START TRANSACTION;

SELECT * FROM students WHERE id = 10 FOR UPDATE;  -- ✗ 阻塞

INSERT INTO students VALUES (20, '新生', 25, 2);  -- ✗ 阻塞

UPDATE students SET age = 19 WHERE id = 15;  -- ✗ 阻塞


-- 整个表都被锁住了!

```


---

## 六、行锁典型应用场景

### 场景描述:用户消费扣款

**问题:**

1. A 用户查询账户余额,判断余额充足
2. 在 A 用户扣款前,B 用户将 A 的余额转走
3. A 用户仍按之前的余额判断执行扣款,导致余额不足但扣款成功

**解决方案:**
查询时使用 `SELECT ... FOR UPDATE` 加排他锁:

```sql
SELECT balance FROM account WHERE user_id = ? FOR UPDATE;
```

这样可以防止其他事务在当前事务完成前修改该记录,确保数据一致性。

---

## 七、总结对比

| 锁类型 | 粒度       | 死锁 | 并发性 | 适用引擎 | 典型场景 |
| ------ | ---------- | ---- | ------ | -------- | -------- |
| 全局锁 | 整个数据库 | 不会 | 极低   | 通用     | 全库备份 |
| 表锁   | 整张表     | 不会 | 低     | MyISAM   | 读多写少 |
| 行锁   | 单行记录   | 可能 | 高     | InnoDB   | 写密集型 |

**核心原则:**

* 锁粒度越小,并发性能越高,但实现复杂度越大
* InnoDB 行锁基于索引实现,必须确保 SQL 走索引
* 可重复读隔离级别下,通过间隙锁和临键锁解决幻读问题



## ✅ 简明结论（先看这个）

不是所有 CRUD 操作都会“自动上锁”，是否上锁、上什么锁， **取决于数据库引擎、事务隔离级别、是否在事务中、以及具体操作类型** 。下面结合你关心的工业实践和 MySQL（尤其是 InnoDB）来清晰解释：

|                                                        |                                    |                                  |
| ------------------------------------------------------ | ---------------------------------- | -------------------------------- |
| **普通 `SELECT`（无事务）**                            | ❌ 不加锁                           | 读的是快照（MVCC），无锁         |
| **`SELECT ... FOR UPDATE`**                            | ✅ 加行级排他锁（X锁）              | 显式加锁，用于悲观并发控制       |
| **`SELECT ... LOCK IN SHARE MODE`** （或 `FOR SHARE`） | ✅ 加行级共享锁（S锁）              | 允许多读，阻塞写                 |
| **`INSERT` / `UPDATE` / `DELETE`（在事务中）**         | ✅ 自动加排他锁（X锁）              | 锁住被修改的行（前提是走索引！） |
| **`INSERT` / `UPDATE` / `DELETE`（自动提交模式）**     | ✅ 加锁，但事务立即提交，锁很快释放 | 仍会加锁，但持续时间极短         |

> 📌  **关键点** ：
>
> * **只有写操作（I/U/D）和显式加锁的读（`FOR UPDATE` 等）才会加锁** 。
> * **普通 `SELECT` 在 InnoDB 中是无锁的** （靠 MVCC 实现一致性读）。




好的！以下是围绕你要求的重点——**第 2 节「事务的生命周期与控制语句」**——进行详细展开，其余部分保持简略。内容结合 MySQL（InnoDB）语法与实际示例，便于理解工业场景中的使用方式。

---

# 10、事务

## 1. 事务的基本概念

事务是数据库操作的逻辑单元，具备 ACID 特性，确保数据在并发和故障场景下依然可靠。

## 2. 事务的生命周期与控制语句 ✅（详细）

事务从**开启**到**结束**（提交或回滚）的全过程，称为其生命周期。MySQL 通过以下 SQL 语句控制事务：

### 2.1 开启事务

有两种方式显式开启事务：

```sql
-- 方式1（推荐）
START TRANSACTION;

-- 方式2（等效）
BEGIN;
```

> 💡 注意：
>
> - 如果 `autocommit = 1`（默认），每条 SQL 自动提交，**不会形成多语句事务**。
> - 使用 `START TRANSACTION` 会**临时关闭 autocommit**，直到 `COMMIT` 或 `ROLLBACK`。

#### ✅ 示例：开启事务并执行操作

```sql
-- 查看当前 autocommit 状态（1=开启，0=关闭）
SHOW VARIABLES LIKE 'autocommit';

-- 显式开启事务
START TRANSACTION;

-- 执行多个操作
UPDATE accounts SET balance = balance - 100 WHERE user_id = 1;
UPDATE accounts SET balance = balance + 100 WHERE user_id = 2;

-- 此时数据已修改，但未持久化（其他会话不可见）
```

---

### 2.2 提交事务（`COMMIT`）

将事务中所有修改**永久写入数据库**，释放所有锁。

```sql
COMMIT;
```

- 一旦提交，无法回滚。
- 其他会话此时才能看到修改结果（取决于隔离级别）。

#### ✅ 接上例：提交转账

```sql
COMMIT;  -- 转账成功，余额变更生效
```

---

### 2.3 回滚事务（`ROLLBACK`）

**撤销**事务中所有已执行的操作，恢复到事务开始前的状态。

```sql
ROLLBACK;
```

- 常用于业务校验失败、异常处理等场景。
- 自动释放事务持有的所有锁。

#### ✅ 示例：余额不足，回滚

```sql
START TRANSACTION;

SELECT balance INTO @bal FROM accounts WHERE user_id = 1;

-- 假设检查发现余额不足
IF @bal < 100 THEN
    ROLLBACK;  -- 取消整个事务
ELSE
    UPDATE accounts SET balance = balance - 100 WHERE user_id = 1;
    UPDATE accounts SET balance = balance + 100 WHERE user_id = 2;
    COMMIT;
END IF;
```

> ⚠️ 注意：MySQL 命令行不支持 `IF`，上述逻辑需在存储过程或应用层实现。

---

### 2.4 保存点（Savepoint）——事务内的“检查点”

允许在**一个事务内设置多个回滚点**，实现部分回滚。

```sql
SAVEPOINT sp_name;                -- 设置保存点
ROLLBACK TO sp_name;              -- 回滚到该点（事务未结束）
RELEASE SAVEPOINT sp_name;        -- 删除保存点（可选）
```

#### ✅ 示例：分阶段操作，部分失败可恢复

```sql
START TRANSACTION;

-- 第一阶段：扣款
UPDATE accounts SET balance = balance - 100 WHERE user_id = 1;
SAVEPOINT after_deduct;

-- 第二阶段：发券（可能失败）
INSERT INTO coupons (user_id, code) VALUES (1, 'SAVE10');
-- 假设这里出错（如唯一约束冲突）

-- 仅回滚发券，保留扣款？不行！应整体回滚或重试
-- 但若设计允许，可：
-- ROLLBACK TO after_deduct;  -- 取消发券，保留扣款（需业务允许！）

-- 最终要么 COMMIT，要么 ROLLBACK
COMMIT;
```

> 🎯 **工业建议**：
> 保存点使用较少，因容易引发业务逻辑混乱。**优先保证事务原子性**，避免“半成功”状态。

---

### 2.5 自动提交模式（`autocommit`）

- 默认 `autocommit = ON`（值为 1）：每条 SQL 自动开启并提交一个事务。
- 设置 `SET autocommit = 0;`：后续 SQL 需手动 `COMMIT`/`ROLLBACK`。

#### ✅ 对比示例：

```sql
-- 默认模式（autocommit=1）
UPDATE accounts SET balance = 1000 WHERE user_id = 1;
-- 立即生效，无法回滚！

-- 手动事务模式
SET autocommit = 0;
UPDATE accounts SET balance = 900 WHERE user_id = 1;
-- 此时未提交，可 ROLLBACK
ROLLBACK;  -- 余额恢复为 1000
SET autocommit = 1;  -- 恢复默认
```

> 💡 **最佳实践**：
> 应用代码中**始终显式使用 `BEGIN`/`COMMIT`**，不要依赖 `autocommit` 控制事务边界，避免歧义。

---

## 3. 事务隔离级别

定义事务之间可见性规则，解决脏读、不可重复读、幻读问题。MySQL InnoDB 默认为 **可重复读（RR）**。

## 4. 事务与锁的关系

写操作（I/U/D）和 `FOR UPDATE` 会加排他锁（X），`FOR SHARE` 加共享锁（S），锁在事务结束时释放。

## 5. MVCC （同上）

InnoDB 通过 undo log 和 read view 实现快照读，使普通 `SELECT` 无锁且高并发。

## 6. 事务日志（下次单开一章节）

Redo Log 保证持久性，Undo Log 支持回滚和 MVCC。

## 7. 并发问题与死锁

死锁由循环等待引起，MySQL 自动检测并回滚代价小的事务；可通过固定访问顺序避免。

## 8. 应用层实践

短事务优于长事务；分布式场景需引入 2PC 或柔性事务（如 Saga）。

## 9. 性能与监控

长事务会阻塞 purge、导致主从延迟；可通过 `innodb_trx` 表监控活跃事务。

## 10. InnoDB 特有机制

RR 隔离级别下通过 **Next-Key Lock**（行锁 + 间隙锁）防止幻读，但仅对 **当前读**（如 `FOR UPDATE`）生效。