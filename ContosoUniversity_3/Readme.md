# EF Core 教程
## 3. 筛选、排序、分页功能
### 排序功能 ###
- 对于一个对象，可以通过不同方法来进行排序，例如对于学生，可以按照姓名升降序或者创建日期升降序排列
- 在Pages/Students/Index.cshtml.cs中添加如下排序方法并替换原有OnGetAsync方法实现排序：
```c#
     // 排序方法
    public string NameSort { get; set; }
    public string DateSort { get; set; }
    // 筛选器
    public string CurrentFilter { get; set; }
    public string CurrentSort { get; set; }
    
    public IList<Student> Students { get;set; }
    
    public async Task OnGetAsync(string sortOrder)
    {
        // 根据入参决定排列方式, 默认包含姓名升降序，创建日期升降序
        NameSort = String.IsNullOrEmpty(sortOrder) ? "name_desc" : "";
        DateSort = sortOrder == "Date" ? "data_desc" : "Date";
    
        IQueryable<Student> studentsIQ = from s in _context.Students select s;
        switch (sortOrder)
        {
            case "name_desc":
                studentsIQ = studentsIQ.OrderByDescending(s => s.LastName);
                break;
            case "Date":
                studentsIQ = studentsIQ.OrderBy(s => s.EnrollmentDate);
                break;
            case "date_desc":
                studentsIQ = studentsIQ.OrderByDescending(s => s.EnrollmentDate);
                break;
            default:
                studentsIQ = studentsIQ.OrderBy(s => s.LastName);
                break;
        }
    
        Students = await studentsIQ.AsNoTracking().ToListAsync();
    }
```
- 并在Students/Index.cshtml中为列表表头LastName与EnrollmentDate添加按钮：
```html
<a asp-page="./Index" asp-route-sortOrder="@Model.NameSort">
    @Html.DisplayNameFor(model => model.Students[0].LastName)
</a>
<a asp-page="./Index" asp-route-sortOrder="@Model.DateSort">
    @Html.DisplayNameFor(model => model.Students[0].EnrollmentDate)
</a>
```
其中通过asp-route-sortOrder = “”来实现对于后端代码sortOrder的赋值，其效果与HTTP中?sortOrder=相同

### 筛选功能 ###
- 更新OnGetAsync方法，增加传入参数searchString，并针对searchString做出操作：
```c#
CurrentFilter = searchString;
if (!String.IsNullOrEmpty(searchString))
{
    // 可将所有参数全转换为大写即可实现不区分大小写测试
    studentsIQ = studentsIQ.Where(s => s.LastName.Contains(searchString)
                           || s.FirstName.Contains(searchString));
    // studentsIQ = studentsIQ.Where(s => s.LastName.ToUpper().Contains(searchString.ToUpper()) || s.FirstName.ToUpper().Contains(searchString.ToUpper()));
}
```
注意：在这里的ToUpper并不会影响查询效率
- 并在前端添加搜索界面：
```html
<form asp-page="./Index" method="get">
    <div class="form-actions no-color">
        <p>
            Find by name:
            <input type="text" name="SearchString" value="@Model.CurrentFilter" />
            <input type="submit" value="Search" class="btn btn-primary" /> |
            <a asp-page="./Index">Back to full List</a>
        </p>
    </div>
</form>
```

### 分页操作 ###
- 在项目文件夹下创建PaginatedList.cs来实现分页功能，其中使用Skip与Take来筛选数据：
```c#
using System;
using System.Collections.Generic;
using System.Linq;
using System.Threading.Tasks;
using Microsoft.EntityFrameworkCore;

namespace ContosoUniversity_3
{
    public class PaginatedList<T> : List<T>
    {
        public int PageIndex { get; private set; }
        public int TotalPages { get; private set; }

        public PaginatedList(List<T> items, int count, int pageIndex, int pageSize)
        {
            PageIndex = pageIndex;
            TotalPages = (int)Math.Ceiling(count / (double)pageSize);

            this.AddRange(items);
        }

        public bool HasPreviousPage
        {
            get
            {
                return (PageIndex > 1);
            }
        }

        public bool HasNextPage
        {
            get
            {
                return (PageIndex < TotalPages);
            }
        }

        public static async Task<PaginatedList<T>> CreateAsync(
            IQueryable<T> source, int pageIndex, int pageSize)
        {
            var count = await source.CountAsync();
            var items = await source.Skip(
                (pageIndex - 1) * pageSize)
                .Take(pageSize).ToListAsync();
            return new PaginatedList<T>(items, count, pageIndex, pageSize);
        }
    }
}
```
- 然后在appsettings.json配置文件中添加"PageSize"并设置默认一页包含条目个数
- 更新Students/Index.cshtml.cs中代码，添加分页功能，包括储存当前页数功能，以及翻页功能：
```c#
using System;
using System.Collections.Generic;
using System.Linq;
using System.Threading.Tasks;
using Microsoft.AspNetCore.Mvc;
using Microsoft.AspNetCore.Mvc.RazorPages;
using Microsoft.EntityFrameworkCore;
using Microsoft.Extensions.Configuration;
using ContosoUniversity;
using ContosoUniversity.Models;

namespace ContosoUniversity.Pages.Students
{
    public class IndexModel : PageModel
    {
        private readonly ContosoUniversity.SchoolContext _context;
        private readonly IConfiguration Configuration;

        public IndexModel(SchoolContext context, IConfiguration configuration)
        {
            _context = context;
            Configuration = configuration;
        }
        //public IndexModel(ContosoUniversity.SchoolContext context)
        //{
        //    _context = context;
        //}
        // 排序方法
        public string NameSort { get; set; }
        public string DateSort { get; set; }
        // 筛选器
        public string CurrentFilter { get; set; }
        public string CurrentSort { get; set; }

        //public IList<Student> Students { get;set; }

        public PaginatedList<Student> Students { get; set; }

        public async Task OnGetAsync(string sortOrder,string currentFilter, string searchString, int? pageIndex)
        {
            CurrentSort = sortOrder;
            // 根据入参决定排列方式, 默认包含姓名升降序，创建日期升降序
            NameSort = String.IsNullOrEmpty(sortOrder) ? "name_desc" : "";
            DateSort = sortOrder == "Date" ? "data_desc" : "Date";

            IQueryable<Student> studentsIQ = from s in _context.Students select s;
            if (searchString != null)
            {
                pageIndex = 1;
            }
            else
            {
                searchString = currentFilter;
            }
            CurrentFilter = searchString;
            if (!String.IsNullOrEmpty(searchString))
            {
                // 可将所有参数全转换为大写即可实现不区分大小写测试
                studentsIQ = studentsIQ.Where(s => s.LastName.Contains(searchString)
                                       || s.FirstName.Contains(searchString));
                // studentsIQ = studentsIQ.Where(s => s.LastName.ToUpper().Contains(searchString.ToUpper()) || s.FirstName.ToUpper().Contains(searchString.ToUpper()));
            }

            switch (sortOrder)
            {
                case "name_desc":
                    studentsIQ = studentsIQ.OrderByDescending(s => s.LastName);
                    break;
                case "Date":
                    studentsIQ = studentsIQ.OrderBy(s => s.EnrollmentDate);
                    break;
                case "date_desc":
                    studentsIQ = studentsIQ.OrderByDescending(s => s.EnrollmentDate);
                    break;
                default:
                    studentsIQ = studentsIQ.OrderBy(s => s.LastName);
                    break;
            }

            var pageSize = Configuration.GetValue("PageSize", 4);
            Students = await PaginatedList<Student>.CreateAsync(
                studentsIQ.AsNoTracking(), pageIndex ?? 1, pageSize);
        }
    }
}

```
- 更新前端网页，为筛选按钮增加存储当前筛选（搜索）功能：     
```html
<a asp-page="./Index" asp-route-sortOrder="@Model.NameSort" asp-route-currentFilter="@Model.CurrentFilter">
    @Html.DisplayNameFor(model => model.Students[0].LastName)
</a>
<a asp-page="./Index" asp-route-sortOrder="@Model.NameSort" asp-route-currentFilter="@Model.CurrentFilter">
    @Html.DisplayNameFor(model => model.Students[0].EnrollmentDate)
</a>
```
- 同时在底端增加翻页功能(若没有上一页或者没有下一页应不能点击上一页或下一页)，同时记录之前的排序方法与筛选输入:     
```html
@{
    var prevDisabled = !Model.Students.HasPreviousPage ? "disabled" : "";
    var nextDisabled = !Model.Students.HasNextPage ? "disabled" : "";
}

<a asp-page="./Index"
   asp-route-sortOrder="@Model.CurrentSort"
   asp-route-pageIndex="@(Model.Students.PageIndex - 1)"
   asp-route-currentFilter="@Model.CurrentFilter"
   class="btn btn-primary @prevDisabled">
    Previous
</a>
<a asp-page="./Index"
   asp-route-sortOrder="@Model.CurrentSort"
   asp-route-pageIndex="@(Model.Students.PageIndex + 1)"
   asp-route-currentFilter="@Model.CurrentFilter"
   class="btn btn-primary @nextDisabled">
    Next
</a>
```

### 分组功能 ###
- 数据库查询可按不同条件进行分组，例如对于学生，可以按照注册日期进行分组    
- 可以创建新的数据模型来生成新的分组视图，在Models/SchoolViewModels中创建EnrollmentDateGroup：
```c#
using System;
using System.ComponentModel.DataAnnotations;

namespace ContosoUniversity.Models.SchoolViewModels
{
    public class EnrollmentDateGroup
    {
        [DataType(DataType.Date)]
        public DateTime? EnrollmentDate { get; set; }

        public int StudentCount { get; set; }
    }
}
```
- 创建Pages/About.cshtml文件：
```html
@page
@model ContosoUniversity.Pages.AboutModel

@{
    ViewData["Title"] = "Student Body Statistics";
}

<h2>Student Body Statistics</h2>

<table>
    <tr>
        <th>
            Enrollment Date
        </th>
        <th>
            Students
        </th>
    </tr>

    @foreach (var item in Model.Students)
    {
        <tr>
            <td>
                @Html.DisplayFor(modelItem => item.EnrollmentDate)
            </td>
            <td>
                @item.StudentCount
            </td>
        </tr>
    }
</table>
```
- 为页面创建About.cshtml.cs文件,使用group by语法将学生用创建日期分组：
```c#
using ContosoUniversity.Models.SchoolViewModels;
using ContosoUniversity.Data;
using Microsoft.AspNetCore.Mvc.RazorPages;
using Microsoft.EntityFrameworkCore;
using System.Collections.Generic;
using System.Linq;
using System.Threading.Tasks;
using ContosoUniversity.Models;

namespace ContosoUniversity.Pages
{
    public class AboutModel : PageModel
    {
        private readonly SchoolContext _context;

        public AboutModel(SchoolContext context)
        {
            _context = context;
        }

        public IList<EnrollmentDateGroup> Students { get; set; }

        public async Task OnGetAsync()
        {
            IQueryable<EnrollmentDateGroup> data =
                from student in _context.Students
                group student by student.EnrollmentDate into dateGroup
                select new EnrollmentDateGroup()
                {
                    EnrollmentDate = dateGroup.Key,
                    StudentCount = dateGroup.Count()
                };

            Students = await data.AsNoTracking().ToListAsync();
        }
    }
}
```