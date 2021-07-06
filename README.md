# Entity Framework Core入门文档 #
***
这是一篇根据[Microsoft官方文档](https://github.com/dotnet/EntityFramework.Docs)
所写的中文帮助文档，并根据所给的例子做出了部分注释。

Entity Framework Core是适用于.NET的新式对象数据库映射器。    

## 版本数据与支持
### 目前支持的数据库 ###
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

## 获取Entity Framework Core运行时
使用Nuget包管理器可以搜索Microsoft.EntityFrameworkCore.SqlServer以获取对应包。
- 此操作在Rider与Visual Studio中均适用。
- 如果不加选择，会默认选择最高最新版本(6.0)，但为稳定，我选择了5.0版本进行测试。

实用的CLI指令：     
```
为.NET添加全局EF Core:
dotnet tool install --global dotnet-ef
```

## 第一个EF Core应用（简单的增删改查调用） 
应用地址: [FirstApp](https://github.com/CaptainDra/EFCoreDemo/tree/master/FirstApp)，这应用可以模拟一个简单的博客的增删改查功能。    
 - 步骤1: 对此应用要作为目标对象的数据库安装EF Core数据库程序包，这个应用使用较为轻量级的SQLite。
 - 步骤2：添加Model.cs, 定义要用的数据结构,并添加增持删改查对应代码至Program.cs中
 - 步骤3: 调试->开始执行(不调试)运行

## EF Core 教程
### 1. EFCore与Razor Pages（轻量级Web界面）的交互 ###
应用地址: [ContosoUniversity_1](https://github.com/CaptainDra/EFCoreDemo/tree/master/ContosoUniversity_1)，通过轻量级web网页开发绑定学生系统并且包括两种不同的数据库初始化方法来初始本地数据库。 

### 2. CRUD操作 ###
应用地址: [ContosoUniversity_2](https://github.com/CaptainDra/EFCoreDemo/tree/master/ContosoUniversity_2)，学生系统的增删改查等功能。 

### 3. 筛选、排序、分页 ###
应用地址: [ContosoUniversity_3](https://github.com/CaptainDra/EFCoreDemo/tree/master/ContosoUniversity_3)，筛选，排序，分页功能。     

### 4. 迁移 ###
应用地址: [ContosoUniversity_4](https://github.com/CaptainDra/EFCoreDemo/tree/master/ContosoUniversity_4)，迁移。 

### 5. 读取，更新plus ###
应用地址: [ContosoUniversity_5](https://github.com/CaptainDra/EFCoreDemo/tree/master/ContosoUniversity_5)，更复杂的模型与读取更新。    

### 6 并发冲突处理 ###
应用地址: [ContosoUniversity_6](https://github.com/CaptainDra/EFCoreDemo/tree/master/ContosoUniversity_6)，并发冲突处理。