# Appsettings

## 1. 创建ConfigServices类
在最终的主程序中调用该设置服务来调用设置好的Appsettings,基础款类如下:
```c#
namespace Core.Extensions
{
    /// <summary>
    /// 读取配置文件
    /// </summary>
    public class ConfigServices
    {
        public static IConfiguration Configuration { get; set; }
        static ConfigServices()
        {
            //ReloadOnChange = true 当appsettings.json被修改时重新加载            
            Configuration = new ConfigurationBuilder()
            .Add(new JsonConfigurationSource { Path = "appsettings.json", ReloadOnChange = true })
            .Build();
        }
    }
}
```

## 2. 在Appsettings.json增加配置
在Appsetting.json/appsetting.Development.json中以json形式增加自定义配置。
例如：数据库连接，数据库对应，访问策略，LDAP对应等信息都可以存在这里，替代原有的webconfig形式的配置文件

### 3. 新建一个读取配置的类
例如如果你有两个数据库(MySQL+SQLServer)对应两个不同的服务，需要配置一个类来获取这个值
```c#
namespace Core.Model.ConfigModel
{
    /// <summary>
    /// 数据库连接字符串
    /// </summary>
    public class DBConnection
    {
        /// <summary>
        /// MySql数据库连接字符串
        /// </summary>
        public string MySqlConnectionString { get; set; }
 
        /// <summary>
        /// SqlServer数据库连接字符串
        /// </summary>
        public string SqlServerConnectionString { get; set; }
    }
}
```
注意，这里的读取可以只有get不用set。