# EF Core 教程
## 4.迁移功能
1. 删除数据库功能：
```PowerShell
Drop-Database
```

2. 创建初始迁移(创建一个名为InitialCreate的迁移文件并更新数据库):
```PowerShell
Add-Migration InitialCreate
Update-Database
```
- 在Migrations/<Time>_InitialCreate.cs中可以看到用于创建数据库的代码，其中Up方法为创建数据表，Down为删除表方法     
    - 其中一般创建数据库代码会以<Time>_<Migration_Name>.cs形式存在Migrations文件夹中
- 同时在迁移过程中会在Migrations/SchoolContextModelSnapshot.cs中生成对应快照，如果想要回滚或返回最近迁移，需要使用migrations remove命令来删除迁移并重置快照


ps.:不建议在生产服务器中使用迁移，容易造成读写冲突，应在部署过程中使用：
```
dotnet ef database update
```