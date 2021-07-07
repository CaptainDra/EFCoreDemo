# EF Core常见基础任务
EF Core是Entity Framework的全新的为.Net Core的定制版本，支持多种版本的数据库，在.Net Core应用中安装EF Core只需要Nuget包安装Microsoft.EntityFrameworkCore包，以及具体数据库提供程序的包。
## 1. 建模数据库
可以通过代码编写模型组类实现数据库支持，也可以通过工具逆向生成这些组类

### 1.1 定义数据库和模型
数据库一般是按照DbContext类派生建模，该类包含多个类型为DbSet<T>的集合属性，连接字符串，对象注入，例如:[示例数据库构造](https://github.com/CaptainDra/EFCoreDemo/blob/master/ContosoUniversity_1/Data/SchoolContext.cs)     
然后表的构造类一般是使用一个类来描述表中关系然后与公共接口匹配，例如:[示例表类型构造](https://github.com/CaptainDra/EFCoreDemo/blob/master/ContosoUniversity_1/Models/Student.cs)    
一般在创建类的时候考虑类的面向对象特征，并且每一个生成表的类必须有一个主键

### 1.2 自动创建数据库
在EF Core中，当数据库不存在需要初始数据库的时候，需要用Database.EnsureCreated()通过调用DbContext类中共有方法来显式创建数据库。
也可以使用SeedTable来生成已经有数据的数据库。

## 2. 表数据处理
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
参考[学生信息](https://github.com/CaptainDra/EFCoreDemo/blob/master/ContosoUniversity_2/Models/Student.cs)，其中的注册信息为外键，
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