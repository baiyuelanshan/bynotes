本篇列举Apache Doris建表语句常用到的参数

#### 1. replication_num

每个 Tablet 的副本数量。默认为3，建议保持默认即可。在建表语句中，所有 Partition 中的 Tablet 副本数量统一指定。而在增加新分区时，可以单独指定新分区中 Tablet 的副本数量。

副本数量可以在运行时修改。强烈建议保持奇数。

最大副本数量取决于集群中独立IP的数量（注意不是BE数量）。Doris中副本分布的原则是，不允许同一个Tablet的副本分布在同一台物理机上，而识别物理机即通过IP。所以，即使在同一台物理机上部署了3个或更多BE实例，如果这些BE的IP相同，则依然只能设置副本数为1。

对于一些小，并且更新不频繁的维度表，可以考虑设置更多的副本数。这样在Join查询时，可以有更大的概率进行本地数据Join。

#### 2. storage_medium & storage_cooldown_time

BE的数据存储目录可以显式的指定为 SSD 或者 HDD（通过 .SSD 或者 .HDD 后缀区分）。建表时，可以统一指定所有 Partition 初始存储的介质。注意，后缀作用是显式指定磁盘介质，而不会检查是否与实际介质类型相符。

默认初始存储介质可通过fe的配置文件 fe.conf 中指定 `default_storage_medium=xxx`，如果没有指定，则默认为HDD。如果指定为 SSD，则数据初始存放在SSD上。

如果没有指定`storage_cooldown_time`，则默认30天后，数据会从SSD自动迁移到HDD 上。如果指定了`storage_cooldown_time`，则在到达`storage_cooldown_time`时间后，数据才会迁移。

注意，当指定storage_medium时，如果FE参数`enable_strict_storage_medium_check`为 False 该参数只是一个“尽力而为”的设置。即使集群内没有设置SSD存储介质，也不会报错，而是自动存储在可用的数据目录中。同样，如果SSD介质不可访问、空间不足，都可能导致数据初始直接存储在其他可用介质上。而数据到期迁移到HDD时，如果HDD介质不可访问、空间不足，也可能迁移失败（但是会不断尝试）。

如果FE参数 `enable_strict_storage_medium_check`为True则当集群内没有设置SSD存储介质时，会报错 Failed to find enough host in all backends with storage medium is SSD。

#### 3. ENGINE

olap，默认的ENGINE类型。在Doris中，只有这个 ENGINE 类型是由Doris负责数据管理和存储的。其他ENGINE类型，如mysql、broker、es等等，本质上只是对外部其他数据库或系统中的表的映射，以保证Doris可以读取这些数据。而Doris本身并不创建、管理和存储任何非olap ENGINE 类型的表和数据。


