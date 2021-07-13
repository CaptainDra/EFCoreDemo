# EF Core常见基础任务
EF Core是Entity Framework的全新的为.Net Core的定制版本，支持多种版本的数据库，在.Net Core应用中安装EF Core只需要Nuget包安装Microsoft.EntityFrameworkCore包，以及具体数据库提供程序的包。
## 1. 建模数据库
可以通过代码编写模型组类实现数据库支持，也可以通过工具逆向生成这些组类    
参考:    
[教程: 数据库建模](../ContosoUniversity_1)     
[教程: 更复杂的数据库建模](../ContosoUniversity_5)      

### 1.1 定义数据库和模型
数据库一般是按照DbContext类派生建模，该类包含多个类型为DbSet<T>的集合属性，连接字符串，对象注入，例如:[示例数据库构造](../ContosoUniversity_1/Data/SchoolContext.cs)     
然后表的构造类一般是使用一个类来描述表中关系然后与公共接口匹配，例如:[示例表类型构造](../ContosoUniversity_1/Models/Student.cs)    
一般在创建类的时候考虑类的面向对象特征，并且每一个生成表的类必须有一个主键

### 1.2 自动创建数据库
在EF Core中，当数据库不存在需要初始数据库的时候，需要用Database.EnsureCreated()通过调用DbContext类中共有方法来显式创建数据库。
也可以使用SeedTable来生成已经有数据的数据库。

## 2. 表数据处理
[教程: 基础增删改查操作](../ContosoUniversity_2)     
[教程: 基于选择的筛选排序分页操作](../ContosoUniversity_3)     
[教程: 复杂数据查询处理](../ContosoUniversity_5)     
### 2.1 查询记录
EF Core微软官方文档建议使用LINQ来查询数据库中的数据。数据库将LINQ查询的表示形式传递给数据库提供程序，然后进行查，例如：    
```c#
using (var context = new BloggingContext())
{
    var blog = context.Blogs
        .Single(b => b.BlogId == 1);
}
```
按照部其他教程所写，也可以使用关系型数据库查询语句，例如：
```c#
using (var context = new BloggingContext())
{
    var blog = (from b in context.Blogs
                    where b.BlogId == '1').FirstOrDefault();
}
```

### 2.2 处理关系
参考[学生信息](../ContosoUniversity_2/Models/Student.cs)，其中的注册信息为外键，
在查询记录的时候可以通过include方法来调用底层的JOIN语句来展开外键内容。   
但是使用的Include方法越多，关联的表就越多占用的内存量也越多。

### 2.3 增加记录
通过代码在内存中增加一个对象，然后把集合持久化到数据库，可通过如下代码：
```c#
using (var context = new BloggingContext())
{
    context.Blogs.Add(newBlog);
    try
    {
        context.SaveChanges();
    }
    catch(...)
    {
        // ...
    }
}
```
来保存数据，需要注意保存数据的时候需要各种验证

### 2.4 更新记录
更新记录分为两个步骤：
1. 查询相关数据
2. 在相同的DbContext中进行修改
```c#
using (var context = new BloggingContext())
{
    // 1 查询
    var blog = context.Blogs
        .Single(b => b.BlogId == 1);
    // 2 修改
    blog.BlogID = 2;
    try
    {
        context.SaveChanges();
    }
    catch(...)
    {
        // ...
    }
}
```
在更新的过程中需要考虑的就是并发冲突的问题，这个会在冲突中继续讨论

### 2.5 删除记录
删除记录也是两步：
1. 查询相关信息
2. 通过Remove方法移除context中的对象。     

但是一般为了维护业务，不会进行删除操作。
   
## 3. 迁移功能
当数据表发生变化，或者模型发生变化时，需要操作统一建模与数据，此时可用到迁移功能      
[教程：迁移](../ContosoUniversity_4)
### 3.1 使用迁移的意义
当数据库表发生变化后（增加或删除实体或属性），需要使应用程序与数据库保持同步，迁移功能可以以递增的方式更新数据库架构，使得程序与数据模型保持同步，同时保留现有数据

### 3.2 创建迁移
使用指令Add-Migration即可创建迁移，使用dotnet ef database update即可根据迁移创建数据库和架构    

### 3.3 迁移与种子文件
Add-Migration操作的同时可以产生种子文件，可以用这个在创建数据库的时候用来当种子文件初始化

## 4. 冲突 & 处理
对于并发操作，会遇到冲突，EF Core提供乐观并发控制解决冲突    
[教程: 并发与冲突处理](../ContosoUniversity_6)
### 4.1 EF Core中的并发冲突
对于数据库中事务处理的冲突，EF Core采用乐观并发控制来处理冲突，每当EF Core调用SaveChanges方法时，会比对一个token值来判断是否存在冲突。(此token会在事务开始时从数据库获取并在数据库更新后更新)：    
- 若该token没变，则事务继续执行
- 若该token不匹配，则证明该数据已经被修改过，事务终止，报错给前台    

在关系型数据库中，EF Core会对Update与Delete操作中的Where子句颁发一个并发token，在执行后会返回影响行数：    
- 一般会把令牌颁发给最常变化且经常用作WHERE查询的列名
- 若返回行数为0，则检测并发冲突，然后抛出DbUpdateConcurrencyException     

### 4.2 处理冲突 
在冲突处理中一般会有三组值：
1. 用户在其事务开始时候从数据库读取的值
2. 用户在其事务中尝试写入数据库的值
3. 数据库中当前的值(被其他用户更改后的值)    

因此对应的冲突处理流程一般如下：
1. SaveChanges方法抛出DbUpdateConcurrencyException冲突
2. 使用DbUpdateConcurrencyException.Entries获得冲突的条目(选择是否显示到前台)
3. 重新刷新事务，获取最新的token并重新获取数据库中的值
4. 请用户重新提交/自动重新提交事务直到没有冲突    

