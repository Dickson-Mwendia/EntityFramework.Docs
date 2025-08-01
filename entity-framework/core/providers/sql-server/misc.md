---
title: Miscellaneous - Microsoft SQL Server Database Provider - EF Core
description: Miscellaneous for the Microsoft SQL Server database provider
author: roji
ms.date: 09/20/2022
uid: core/providers/sql-server/misc
---
# Miscellaneous notes for SQL Server

## Azure SQL database options

> [!NOTE]
> You should [use `UseAzureSql` method](xref:core/providers/sql-server/index#usage) instead of `UseSqlServer` when connecting to Azure SQL.

Azure SQL Database provides [a variety of pricing options](https://azure.microsoft.com/pricing/details/sql-database/single/) that are usually configured through the Azure Portal. However, if you are managing the schema using [EF Core migrations](xref:core/managing-schemas/migrations/index), you can configure the desired options with EF, and they will get applied when EF creates the database.

You can specify the service tier of the database (EDITION) using [HasServiceTier](/dotnet/api/Microsoft.EntityFrameworkCore.SqlServerModelBuilderExtensions.HasServiceTier):

[!code-csharp[HasServiceTier](../../../../samples/core/SqlServer/AzureDatabase/AzureSqlContext.cs?name=HasServiceTier)]

You can specify the maximum size of the database using [HasDatabaseMaxSize](/dotnet/api/Microsoft.EntityFrameworkCore.SqlServerModelBuilderExtensions.HasDatabaseMaxSize):

[!code-csharp[HasDatabaseMaxSize](../../../../samples/core/SqlServer/AzureDatabase/AzureSqlContext.cs?name=HasDatabaseMaxSize)]

You can specify the performance level of the database (SERVICE_OBJECTIVE) using [HasPerformanceLevel](/dotnet/api/Microsoft.EntityFrameworkCore.SqlServerModelBuilderExtensions.HasPerformanceLevel):

[!code-csharp[HasPerformanceLevel](../../../../samples/core/SqlServer/AzureDatabase/AzureSqlContext.cs?name=HasPerformanceLevel)]

Use [HasPerformanceLevelSql](/dotnet/api/Microsoft.EntityFrameworkCore.SqlServerModelBuilderExtensions.HasPerformanceLevelSql) to configure the elastic pool, since the value is not a string literal:

[!code-csharp[HasPerformanceLevel](../../../../samples/core/SqlServer/AzureDatabase/AzureSqlContext.cs?name=HasPerformanceLevelSql)]

> [!TIP]
> You can find all the supported values in the [ALTER DATABASE documentation](/sql/t-sql/statements/alter-database-transact-sql?view=azuresqldb-current&preserve-view=true).

## SaveChanges, database triggers and unsupported computed columns

Starting with EF Core 7.0, EF Core saves changes to the database with significantly optimized SQL; unfortunately, this technique is not supported on SQL Server if the target table has database triggers, or certain kinds of computed columns. For more information on this SQL Server limitation, see the documentation on the [OUTPUT clause](/sql/t-sql/queries/output-clause-transact-sql#remarks).

You can let EF Core know that the target table has a trigger; doing so will revert to the previous, less efficient technique. This can be done by configuring the corresponding entity type as follows:

[!code-csharp[Main](../../../../samples/core/SqlServer/Misc/TriggersContext.cs?name=TriggerConfiguration&highlight=4)]

Note that doing this doesn't actually make EF Core create or manage the trigger in any way - it currently only informs EF Core that triggers are present on the table. As a result, any trigger name can be used, and this can also be used if an unsupported computed column is in use (regardless of triggers).

If most or all of your tables have triggers, you can opt out of using the newer, efficient technique for all your model's tables by using the following [model building convention](xref:core/modeling/bulk-configuration#conventions):

[!code-csharp[Main](../../../../samples/core/SqlServer/Misc/TriggersContext.cs?name=BlankTriggerAddingConvention)]

Use the convention on your `DbContext` by overriding `ConfigureConventions`:

[!code-csharp[Main](../../../../samples/core/SqlServer/Misc/TriggersContext.cs?name=ConfigureConventions)]

This effectively calls `HasTrigger` on all your model's tables, instead of you having to do it manually for each and every table.
