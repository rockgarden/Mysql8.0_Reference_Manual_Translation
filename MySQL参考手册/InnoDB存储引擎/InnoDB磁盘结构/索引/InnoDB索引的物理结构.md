# InnoDB索引的物理结构

除了空间索引，InnoDB 索引都是 B-tree 数据结构。空间索引使用 R-tree，它是用于索引多维数据的专用数据结构。索引记录存储在其 B 树或 R 树数据结构的叶页中。索引页的默认大小为 16KB。页面大小由 MySQL 实例初始化时的 innodb_page_size 设置决定。请参阅[第 15.8.1 节，“InnoDB 启动配置”](https://dev.mysql.com/doc/refman/8.0/en/innodb-init-startup-configuration.html)。

当新记录插入到 InnoDB 聚集索引中时，InnoDB 会尝试留出 1/16 的页面空闲以供将来插入和更新索引记录。如果索引记录按顺序（升序或降序）插入，则生成的索引页大约是 15/16 满。如果以随机顺序插入记录，则页面从 1/2 到 15/16 满。

InnoDB 在创建或重建 B 树索引时执行批量加载。这种创建索引的方法称为排序索引构建。 innodb_fill_factor 变量定义在排序索引构建期间填充的每个 B 树页面上的空间百分比，剩余空间保留用于未来索引增长。空间索引不支持排序索引构建。有关更多信息，请参阅[第 15.6.2.3 节，“排序索引构建”](https://dev.mysql.com/doc/refman/8.0/en/sorted-index-builds.html)。将 innodb_fill_factor 设置为 100 后，聚集索引页中 1/16 的空间可用于未来的索引增长。

如果 InnoDB 索引页面的填充因子低于 MERGE_THRESHOLD（如果未指定，则默认为 50%），InnoDB 会尝试收缩索引树以释放页面。 MERGE_THRESHOLD 设置适用于 B-tree 和 R-tree 索引。有关详细信息，请参阅[第 15.8.11 节，“为索引页配置合并阈值”](https://dev.mysql.com/doc/refman/8.0/en/index-page-merge-threshold.html)。
