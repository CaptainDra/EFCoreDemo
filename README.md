# Entity Framework Core入门文档 #
***
这是一篇根据[Microsoft官方文档](https://github.com/dotnet/EntityFramework.Docs)
所写的中文帮助文档，并根据所给的例子做出了部分注释。

Entity Framework Core是适用于.NET的新式对象数据库映射器。    

## 版本数据与支持
###目前支持的数据库
适用数据库：SQL数据库（本地和Azure）、SQLite、MySQL、PostgreSQL、Azure Cosmos DB     
[数据库支持具体版本](https://docs.microsoft.com/zh-cn/ef/core/providers/?tabs=dotnet-core-cli)     
### 目标支持的.NET版本
[官方支持网页](https://docs.microsoft.com/zh-cn/ef/core/what-is-new/)     

| EF Core版本 | 目标Standard版本 | 支持截止时间（预计） |   
| ---------- | --------------- | ---------------- | 
|5.0         |2.1              |2022.2            |
|3.1         |2.0              |2022.12.3  长期支持 |
|3.0         |2.1              |20203.3.3         |
|2.2         |2.0              |2019.12.23 已过期  |
|2.1         |2.0              |2021.8.21  长期支持 |
|2.0         |2.0              |2018.10.1         |
|1.1         |1.3              |2019.6.27         |
|1.0         |1.3              |2019.6.27         |

注： 下一个计划稳定版本为6.0 计划发布于2021.11（当前正以[每日生成](https://github.com/dotnet/aspnetcore/blob/main/docs/DailyBuilds.md)方式在GitHub更新）    

## API参考
参考接口可见[帮助文档](https://docs.microsoft.com/zh-cn/dotnet/api/)     

## 基础入门教程

官方训练于适用: [使用 Entity Framework Core 保留和检索关系数据](https://docs.microsoft.com/zh-cn/learn/modules/persist-data-ef-core/)