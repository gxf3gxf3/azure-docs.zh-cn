---
title: "将数据载入 Azure SQL 数据仓库 |Microsoft 文档"
description: "了解将数据载入 SQL 数据仓库的常见方案。 这些常见方案包括使用 PolyBase、Azure Blob 存储、平面文件以及磁盘寄送。 也可使用第三方工具。"
services: sql-data-warehouse
documentationcenter: NA
author: barbkess
manager: jhubbard
editor: 
ms.assetid: 2253bf46-cf72-4de7-85ce-f267494d55fa
ms.service: sql-data-warehouse
ms.devlang: NA
ms.topic: article
ms.tgt_pltfrm: NA
ms.workload: data-services
ms.date: 10/31/2016
ms.author: barbkess
translationtype: Human Translation
ms.sourcegitcommit: 2ea002938d69ad34aff421fa0eb753e449724a8f
ms.openlocfilehash: eae347c75ddaadcf41644f80bcdc5880ee94541a


---
# <a name="load-data-into-azure-sql-data-warehouse"></a>将数据载入 Azure SQL 数据仓库
本文汇总了将数据载入 SQL 数据仓库时的方案选项和建议。

加载数据时，最难的部分通常是准备要加载的数据。 Azure 使用 Azure Blob 存储作为许多此类服务的常用数据存储，并使用 Azure 数据工厂来协调 Azure 服务之间的通信和数据移动，从而简化了加载过程。 这些过程集成了 PolyBase 技术，因此可以通过大规模并行处理 (MPP) 将数据从 Azure Blob 存储并行载入 SQL 数据仓库。 

有关加载示例数据库的教程，请参阅[加载示例数据库][加载示例数据库]。

## <a name="load-from-azure-blob-storage"></a>从 Azure Blob 存储加载
将数据导入 SQL 数据仓库时，最快的方式是使用 PolyBase 从 Azure Blob 存储加载数据。 PolyBase 使用 SQL 数据仓库的大规模并行处理 (MPP) 设计以并行方式从 Azure Blob 存储加载数据。 若要使用 PolyBase，可以使用 T-SQL 命令或 Azure 数据工厂管道。

### <a name="1-use-polybase-and-t-sql"></a>1.使用 PolyBase 和 T-SQL
加载过程摘要：

1. 将数据格式化为 UTF-8，因为 PolyBase 目前不支持 UTF-16。
2. 将数据移到 Azure Blob 存储并存储在文本文件中。
3. 在 SQL 数据仓库中配置外部对象，以便定义数据的位置和格式
4. 运行 T-SQL 命令，将数据以并行方式载入新的数据库表。

<!-- 5. Schedule and run a loading job. --> 

有关本教程的信息，请参阅[将数据从 Azure Blob 存储载入 SQL 数据仓库 (PolyBase)][将数据从 Azure Blob 存储载入 SQL 数据仓库 (PolyBase)]。

### <a name="2-use-azure-data-factory"></a>2.使用 Azure 数据工厂
如需简化 PolyBase 使用方式，可创建一个 Azure 数据工厂管道，以便使用 PolyBase 将数据从 Azure Blob 存储载入 SQL 数据仓库。 这配置起来很快速，因为不需定义 T-SQL 对象。 若需在不导入的情况下查询外部数据，请使用 T-SQL。 

加载过程摘要：

1. 将数据格式化为 UTF-8，因为 PolyBase 目前不支持 UTF-16。
2. 将数据移到 Azure Blob 存储并存储在文本文件中。
3. 创建 Azure 数据工厂管道，以便引入数据。 使用 PolyBase 选项。
4. 计划和运行管道。

有关本教程的信息，请参阅[将数据从 Azure Blob 存储载入 SQL 数据仓库（Azure 数据工厂）][将数据从 Azure Blob 存储载入 SQL 数据仓库（Azure 数据工厂）]。

## <a name="load-from-sql-server"></a>从 SQL Server 加载
若要将数据从 SQL Server 加载到 SQL 数据仓库，你可以使用 Integration Services (SSIS)，可以传输平面文件，还可以将磁盘寄送到 Microsoft。 继续阅读下去，你会看到各种不同加载过程的摘要，以及教程链接。

若打算将数据从 SQL Server 完整迁移到 SQL 数据仓库，请参阅[迁移概述][迁移概述]。 

### <a name="use-integration-services-ssis"></a>使用 Integration Services (SSIS)
如果你使用的已经是要载入 SQL Server 的 Integration Services (SSIS) 包，则可对这些包进行更新，以便将 SQL Server 用作源，将 SQL 数据仓库用作目标。 如果你不打算对加载过程进行迁移以便使用云中的已有数据，则这种方式执行起来既快速又方便。 不利的一面是，加载速度会比使用 PolyBase 慢，因为此 SSIS 不会以并行方式进行加载。

加载过程摘要：

1. 修改 Integration Services 包，使之指定 SQL Server 实例作为源，指定 SQL 数据仓库数据库作为目标。
2. 将架构迁移到 SQL 数据仓库（如果尚未迁移到该处）。
3. 更改包中的映射方式，仅使用 SQL 数据仓库支持的数据类型。
4. 计划和运行包。

有关本教程的信息，请参阅[将数据从 SQL Server 载入 Azure SQL 数据仓库 (SSIS)][将数据从 SQL Server 载入 Azure SQL 数据仓库 (SSIS)]。

### <a name="use-azcopy-recommended-for-10-tb-data"></a>使用 AZCopy（建议用于数据量 < 10 TB 的情况）
如果数据大小 < 10 TB，则可先将数据从 SQL Server 导出到平面文件，然后将这些文件复制到 Azure Blob 存储，最后再使用 PolyBase 将数据载入 SQL 数据仓库

加载过程摘要：

1. 使用 bcp 命令行实用程序将数据从 SQL Server 导出到平面文件。
2. 使用 AZCopy 命令行实用程序将数据从平面文件复制到 Azure Blob 存储。
3. 通过 PolyBase 载入到 SQL 数据仓库中。

有关本教程的信息，请参阅[将数据从 Azure Blob 存储载入 SQL 数据仓库 (PolyBase)][将数据从 Azure Blob 存储载入 SQL 数据仓库 (PolyBase)]。

### <a name="use-bcp"></a>使用 bcp
如果你的数据量小，则可使用 bcp 将数据直接载入 Azure SQL 数据仓库。

加载过程摘要：

1. 使用 bcp 命令行实用程序将数据从 SQL Server 导出到平面文件。
2. 使用 bcp 将数据从平面文件直接加载到 SQL 数据仓库。

有关本教程的信息，请参阅[将数据从 SQL Server 载入 Azure SQL 数据仓库 (bcp)][将数据从 SQL Server 载入 Azure SQL 数据仓库 (bcp)]。

### <a name="use-importexport-recommended-for-10-tb-data"></a>使用导入/导出（建议用于数据量 > 10 TB 的情况）
如果你的数据大小大于 10 TB，并且需要将数据移至 Azure，则建议使用磁盘寄送服务：[导入/导出][导入/导出]。 

加载过程摘要

1. 使用 bcp 命令行实用程序将数据从 SQL Server 导出到可转移磁盘上的平面文件。
2. 将磁盘寄送到 Microsoft。
3. Microsoft 将数据载入 SQL 数据仓库

## <a name="load-from-hdinsight"></a>从 HDInsight 加载
SQL 数据仓库支持通过 PolyBase 从 HDInsight 加载数据。 该过程和从 Azure Blob 存储加载数据一样 - 使用 PolyBase 连接到 HDInsight 以加载数据。 

### <a name="1-use-polybase-and-t-sql"></a>1.使用 PolyBase 和 T-SQL
加载过程摘要：

1. 将数据格式化为 UTF-8，因为 PolyBase 目前不支持 UTF-16。
2. 将数据移到 HDInsight，并将其存储为文本文件、ORC 或 Parquet 格式。
3. 在 SQL 数据仓库中配置外部对象，以便定义数据的位置和格式。
4. 运行 T-SQL 命令，将数据以并行方式载入新的数据库表。

有关本教程的信息，请参阅[将数据从 Azure Blob 存储载入 SQL 数据仓库 (PolyBase)][将数据从 Azure Blob 存储载入 SQL 数据仓库 (PolyBase)]。

## <a name="recommendations"></a>建议
我们的很多合作伙伴都提供加载解决方案。 若要获取更多解决方案，请参阅我们的[解决方案合作伙伴][解决方案合作伙伴]的列表。 

如果你的数据来自非关系源，而你想要将其载入 SQL 数据仓库，则需在加载前先将其转换成行和列。 已转换的数据不需存储在数据库中，可以存储在文本文件中。

基于新加载的数据创建统计信息。 Azure SQL 数据仓库尚不支持自动创建或自动更新统计信息。  为了获得查询的最佳性能，在首次加载数据或者在数据发生重大更改之后，创建所有表的所有列统计信息非常重要。  有关详细信息，请参阅[统计信息][统计信息]。

## <a name="next-steps"></a>后续步骤
有关更多开发技巧，请参阅[部署概述][部署概述]。

<!--Image references-->

<!--Article references-->
[将数据从 Azure Blob 存储载入 SQL 数据仓库 (PolyBase)]: ./sql-data-warehouse-load-from-azure-blob-storage-with-polybase.md
[将数据从 Azure Blob 存储载入 SQL 数据仓库（Azure 数据工厂）]: ./sql-data-warehouse-load-from-azure-blob-storage-with-data-factory.md
[将数据从 SQL Server 载入 Azure SQL 数据仓库 (SSIS)]: ./sql-data-warehouse-load-from-sql-server-with-integration-services.md
[将数据从 SQL Server 载入 Azure SQL 数据仓库 (bcp)]: ./sql-data-warehouse-load-from-sql-server-with-bcp.md
[将数据从 SQL Server 载入 Azure SQL 数据仓库 (AZCopy)]: ./sql-data-warehouse-load-from-sql-server-with-azcopy.md

[加载示例数据库]: ./sql-data-warehouse-load-sample-databases.md
[迁移概述]: ./sql-data-warehouse-overview-migrate.md
[解决方案合作伙伴]: ./sql-data-warehouse-partner-business-intelligence.md
[部署概述]: ./sql-data-warehouse-overview-develop.md
[统计信息]: ./sql-data-warehouse-tables-statistics.md

<!--MSDN references-->

<!--Other Web references-->
[导入/导出]: https://azure.microsoft.com/documentation/articles/storage-import-export-service/



<!--HONumber=Nov16_HO3-->

