# EF Core 教程
## 5.复杂功能
### 更新数据模型 ###
- 为满足更繁琐的读取功能，更新数据模型，为课程添加所属部门-讲师-办公室的数据模型
- 首先更新Student.cs中代码（增加一个全名 = 姓 + 名的字符串属性），并为之前已创建过的参数添加限定条件：
```c#
[DataType] // 数据类型 形如[DataType(DataType.Dtae)]
[DisplayFormat] // 动态显示格式 形如 [DisplayFormat(DataFormatString = "{0:yyyy-MM-dd}", ApplyFormatInEditMode = true)]
[StringLength] // 长度（比如限定姓名不能长于50字符） [StringLength(50, ErrorMessage = "First name cannot be longer than 50 characters.")]
[Column] // 将属性映射到数据库列（用于处理模型中名称与数据库中名称不一致的情况）
[Required] // 是否必填
[Display] //  为实体中类型或成员指定可本地化字符串的通用特性，一般用于指定文本框标题栏
```
- 然后添加讲师(Instructor)实体，并为其添加课程与办公室两个导航属性，其中一名讲师可以对应多个课程，因此将Courses定义为集合，但是导师与办公室应为一一对应的（甚至可以为空）     
- 然后定义办公室注册(OfficeAssignment)实体，需要注意的是这并不是一个办公室实体，而是一个教师对应办公室的实体，所以主键为讲师ID：    
  - 其中讲师中的Instructor.OfficeAssignment可以为空，因为讲师可以没有办公室，所以就没有对应的办公室注册实体。    
  - 而OfficeAssignment.Intructor 不能为空，因为存在办公室注册对应的时候证明这个导师拥有办公室，并且InstructorID为int型不能为空
```c#
[Key] // 当主键名称不为ID或classnameID的时候用于声明主键，例如OfficeAssignment的主键为InstructorID
```
- 更新Course.cs,其中课程应该包含三个导航属性：
  - 对于多个学生注册信息的注册集合
  - 对于一个系的DepartmentID(外键)， Department导航属性(一对一)
  - 一门课可能有多位讲师，所以讲师也应该是一对多集合的导航属性
- 创建Department.cs
  - 其中经费在c#中为decimal类型，但是在数据库中存储为money类型会更好，所以可以使用[Column(TypeName="money")]来创建声明
  - 一个系应该有管理员，但是可以没有，所以选择一个讲师作为管理员，这个讲师id可以为空所以使用int?来声明
  - 一个系可以有多门课程，所以使用一对多来导航
- 更新Enrollment.cs
  - 为分数(Grade)增加一个显示格式[DisplayFormat(NullDisplayText = "No grade")]，当分数为空时显示“没有分数”    
  - 外键有CourseID + Course和StudentID + Student
  - Enrollment的存在意义：因为Student与Course的实体关系为多对多，所以Enrollment充当数据库中的多对多连接表，将多对多关系转化为一对多+一对多关系
- 更新数据库上下文(Data/SchoolContext.cs)
  - 配置讲师与课程多对多关系，并配置实体系部门
- 更新数据库种子(Data/DbInitializer.cs)
- 删除并重建数据库(同4中操作)
- 运行，验证数据库是否更新

### 读取相关数据 ###
- EF Core有多种加载数据的方法(见文档：)
- 创建课程页  
  - 用创建学生基架的方搭建课程基架(模型类用Course，然后使用现有上下文类)
  - 更新Pages/Courses/Index.cshtml.cs与Pages/Courses/Index.cshtml中代码，显示院系名称(使用include方法读取数据，不额外创建视图类)
- 创建讲师页
  - 创建视图模型类(SchoolViewModels/InstructorIndexData.cs),其中包含导师表，课表，注册表
  - 创建讲师页面基架(同上)
  - 更新Pages/Instructors/Index.cshtml.cs中代码，使用视图模型类的数据来按照办公室注册信息-课程-系别加载顺序来获取导师信息，若找到相关导师信息，来用该导师反推导师信息，给界面赋值。
  - 更新Pages/Instructors/Index.cshtml，将界面url查询字符串改为路由数据，添加仅在办公室信息不为null时才显示的办公室列，添加课程列表，添加选课功能等功能
  
### 更新相关数据 ###
- 尝试创建和编辑课程相关数据，先增加一个方法基类Pages/Courses/DepartmentNamePageModel.cs，提供一种可以选择列表的方法
  - 更新课程创建页模型(Pages/Courses/Create.cshtml.cs)，修改派生类，改为派生自DepartmentNamePageModel，使用TryUpdateModelAsync方法更新数据
  - 更新前端界面(Pages/Courses/Create.cshtml),将原本的系id显示改为系名称显示，同时增加未选择系的验证信息。
  - 同样方法更新课程编辑页模型(Pages/Courses/Edit.cshtml.cs)与前端界面(Pages/Courses/Edit.cshtml)，这里不应该能够修改id，所以写死
  