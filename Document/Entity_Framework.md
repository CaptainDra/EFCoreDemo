# Entity Framework
## 简介
Entity Framework是一种老版本的O/RM框架，在.NET应用中承担数据访问任务。但是最新版本的EF6仅仅与.NET Core平台部分兼容。
但是在.NET Core平台中使用EF6必须要完整编译代码，因此并不能支持一个完整的.NET Core应用。并且EF6不支持跨平台功能，仅支持在Windows上运行。

## .NET Core中EF6使用方法
### 1. 将EF6代码封装到独立类库中 ###
因为.Net Core项目并不支持EF6上下文类中能够通过代码触发的全部功能，因此其不支持直接使用EF6中的上下文类。
若一定要在.NET Core程序中使用EF6，可以将所有的类，以及数据库上下文和实体放到一个独立的类库项目中，使其形成完整的框架。   
然后在新的.NET Core项目中添加对于这个独立库项目的引用。

### 2. 连接字符串
EF6中获取连接字符串的方式与.Net Core不完全兼容，一般需要考虑通过参数接受连接字符串/web.config配置(但是.Net Core中没有web.config)    
所以可以使用如下代码将连接字符串声明为常量或者从.Net Core 配置层读取传入：
```c#
public class MyOwnDatabase : DbContext
{
    public MyOwnDatabase(string connStringOrDbName = "name=MyOwnDatabase")
        : base(connStringOrDbName)
    {
    
    }
}
```

### 3. 将EF上下文与.Net Core DI集成
许多文章和文档中建议为DI系统通过依赖注入的方法注入EF6上下文。但是这会导致整个系统效率降低，导致控制器变得臃肿并产生很多的代码层（从输入到数据访问级别），还不如在存储库类中使用DI模式。
