---
title: DM 配置优化
summary: 介绍如何通过优化配置来提高数据迁移性能。
aliases: ['/docs-cn/tidb-data-migration/dev/tune-configuration/','/zh/tidb-data-migration/dev/tune-configuration/']
---

# DM 配置优化

本文档介绍如何对迁移任务的配置进行优化，从而提高 DM 的数据迁移性能。

## 全量导出

全量导出相关的配置项为 `mydumpers`，下面介绍和性能相关的参数如何配置。

### `rows`

设置 `rows` 选项可以开启单表多线程并发导出，值为导出的每个 chunk 包含的最大行数。开启后，DM 会在 MySQL 的单表并发导出时，优先选出一列做拆分基准，选择的优先级为主键 > 唯一索引 > 普通索引，选出目标列后需保证该列为整数类型（如 `INT`、`MEDIUMINT`、`BIGINT` 等）。

`rows` 的值可以设置为 10000，具体设置的值可以根据表中包含数据的总行数以及数据库的性能做调整。另外也需要设置 `threads` 来控制并发线程数量，默认值为 4，可以适当做些调整。

### `chunk-filesize`

DM 全量备份时会根据 `chunk-filesize` 参数的值把每个表的数据划分成多个 chunk，每个 chunk 保存到一个文件中，大小约为 `chunk-filesize`。根据这个参数把数据切分到多个文件中，这样就可以利用 DM Load 处理单元的并行处理逻辑提高导入速度。该参数默认值为 64（单位为 MB），正常情况下不需要设置，也可以根据全量数据的大小做适当的调整。

> **注意：**
>
> - `mydumpers` 的参数值不支持在迁移任务创建后更新，所以需要在创建任务前确定好各个参数的值。如果需要更新，则需要使用 dmctl stop 任务后更新配置文件，然后再重新创建任务。
> - `mydumpers`.`threads` 可以使用配置项 `mydumper-thread` 替代来简化配置。
> - 如果设置了 `rows`，DM 会忽略 `chunk-filesize` 的值。

## 全量导入

全量导入相关的配置项为 `loaders`，下面介绍和性能相关的参数如何配置。

### `pool-size`

`pool-size` 为 DM Load 阶段线程数量的设置，默认值为 16，正常情况下不需要设置，也可以根据全量数据的大小以及数据库的性能做适当的调整。

> **注意：**
>
> - `loaders` 的参数值不支持在迁移任务创建后更新，所以需要在创建任务前确定好各个参数的值。如果需要更新，则需要使用 dmctl stop 任务后更新配置文件，然后再重新创建任务。
> - `loaders`.`pool-size` 可以使用配置项 `loader-thread` 替代来简化配置。

## 增量复制

增量复制相关的配置为 `syncers`，下面介绍和性能相关的参数如何配置。

### `worker-count`

`worker-count` 为 DM Sync 阶段并发迁移 DML 的线程数量设置，默认值为 16，如果对迁移速度有较高的要求，可以适当调高改参数的值。

### batch

`batch` 为 DM Sync 阶段迁移数据到下游数据库时，每个事务包含的 DML 的数量，默认值为 100，正常情况下不需要调整。

> **注意：**
>
> - `syncers` 的参数值不支持在迁移任务创建后更新，所以需要在创建任务前确定好各个参数的值。如果需要更新，则需要使用 dmctl stop 任务后更新配置文件，然后再重新创建任务。
> - `syncers`.`worker-count` 可以使用配置项 `syncer-thread` 替代来简化配置。
> - `worker-count` 和 `batch` 的设置需要根据实际的场景进行调整，例如：DM 到下游数据库的网络延迟较高，可以适当调高 `worker-count`，调低 `batch`。
