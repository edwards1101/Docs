---
title: Razor syntax reference for ASP.NET Core
author: rick-anderson
description: Learn about Razor markup syntax for embedding server-based code into webpages.
manager: wpickett
ms.author: riande
ms.date: 10/18/2017
ms.prod: asp.net-core
ms.technology: aspnet
ms.topic: article
uid: mvc/views/razor
---
# ASP.NET Core 的 Razor 语法

By [Rick Anderson](https://twitter.com/RickAndMSFT), [Luke Latham](https://github.com/guardrex), [Taylor Mullen](https://twitter.com/ntaylormullen), and [Dan Vicarel](https://github.com/Rabadash8820)

Razor是一种在web页面中嵌入服务器端代码的标记语言。Razor语法由Razor标记，C#和HTML组成。包含Razor的文件一般以 *.cshtml* 为扩展名。

## HTML渲染

Razor 的默认语言是HTML。渲染Razor标记和渲染HTML文件中的HTML是一样的。 *.cshtml* Razor 文件中的HTML标记由服务器原样渲染。

## Razor 语法

Razor支持C#并使用`@`符号从HTML转换到C#。Razor运算并在HTML输出中渲染C#表达式。

当一个`@`符号后面跟随 [Razor 保留关键字](#razor-reserved-keywords) 时，标记被转换为特定Razor标记。如果`@`后跟随的不是Razor保留关键字，标记被转换为纯C#。

在Razor标记中转义`@`符可以使用两个`@`：

```cshtml
<p>@@Username</p>
```

以上代码被渲染为单个`@`符号：

```html
<p>@Username</p>
```

HTML属性和内容中email地址包含的`@`符号不被视为Razor转换字符。Razor解析时不会修改下面例子中的email地址：

```cshtml
<a href="mailto:Support@contoso.com">Support@contoso.com</a>
```

## 隐式 Razor 表达式

`@`后面跟随C#代码构成隐式Razor表达式：

```cshtml
<p>@DateTime.Now</p>
<p>@DateTime.IsLeapYear(2016)</p>
```

除了C#的`await`关键字，隐式表达式一定不能包含空格。如果C#语句有明确的结尾，语句中间可以有空格：

```cshtml
<p>@await DoSomething("hello", "world")</p>
```

隐式表达式 **不能** 包含C#泛型，因为在(`<>`)括号中的字符被解释为HTML标签。下面的代码是 **无效** 的：

```cshtml
<p>@GenericMethod<int>()</p>
```

上面的代码会产生类似如下的编译错误：

 * The "int" element wasn't closed. All elements must be either self-closing or have a matching end tag.
 *  Cannot convert method group 'GenericMethod' to non-delegate type 'object'. Did you intend to invoke the method?

泛型方法必须在 [显式 Razor 表达式](#显式 Razor 表达式) 或者 [Razor 代码块](#razor-code-blocks) 中调用。

## 显式 Razor 表达式

Razor显式表达式由`@`和配对的小括号组成。如，要渲染上周此时的时间可以使用下面的Razor标记：

```cshtml
<p>Last week this time: @(DateTime.Now - TimeSpan.FromDays(7))</p>
```

在`@()`中插入的任何内容都会被计算并渲染到输出。

前面章节描述的隐式表达式一般不能包含空格。如下代码不是从当前时间减去1周：

[!code-cshtml[Main](razor/sample/Views/Home/Contact.cshtml?range=17)]

代码被渲染为如下HTML：

```html
<p>Last week: 7/7/2016 4:39:52 PM - TimeSpan.FromDays(7)</p>
```

显式表达式可以用文本和表达式的结果串接：

```cshtml
@{
    var joe = new Person("Joe", 33);
}

<p>Age@(joe.Age)</p>
```

如果不用显式表达式，`<p>Age@joe.Age</p>`被视为一个email地址，会被渲染成 `<p>Age@joe.Age</p>`。而写成显式表达式会被渲染成`<p>Age33</p>`。


显式表达式可以用来将 *.cshtml* 文件中的泛型方法渲染输出。在隐式表达式中，括号(`<>`)中的字符被解释为HTML标签。下面的标记是 **无效的** Razor：

```cshtml
<p>@GenericMethod<int>()</p>
```

上面的代码会产生类似如下的编译错误：

 * The "int" element wasn't closed. All elements must be either self-closing or have a matching end tag.
 *  Cannot convert method group 'GenericMethod' to non-delegate type 'object'. Did you intend to invoke the method?
 
 下面的代码用显式表达式示例正确的实现方法：

```cshtml
<p>@(GenericMethod<int>())</p>
```

## 表达式编码

鉴定为字符串的C#表达式被HTML编码。鉴定为`IHtmlContent`的C#表达式通过`IHtmlContent.WriteTo`被直接渲染。没有被鉴定为`IHtmlContent` 的C#表达式由`ToString`转换为字符串并在渲染前编码。

```cshtml
@("<span>Hello World</span>")
```

代码被渲染为如下HTML:

```html
&lt;span&gt;Hello World&lt;/span&gt;
```

HTML在浏览器中显示为：

```
<span>Hello World</span>
```

`HtmlHelper.Raw` 输出不被编码而是渲染为HTML标记。

> [!注意]
> 对未消毒的用户输入使用`HtmlHelper.Raw`有安全风险。用户的输入可能包含恶意Javascript或其它攻击。对用户的输入消毒是困难的。避免对用的户输入使用`HtmlHelper.Raw`。

```cshtml
@Html.Raw("<span>Hello World</span>")
```

代码渲染的HTML：

```html
<span>Hello World</span>
```

## Razor 代码块

Razor code blocks start with `@` and are enclosed by `{}`. Unlike expressions, C# code inside code blocks isn't rendered. Code blocks and expressions in a view share the same scope and are defined in order:

```cshtml
@{
    var quote = "The future depends on what you do today. - Mahatma Gandhi";
}

<p>@quote</p>

@{
    quote = "Hate cannot drive out hate, only love can do that. - Martin Luther King, Jr.";
}

<p>@quote</p>
```

The code renders the following HTML:

```html
<p>The future depends on what you do today. - Mahatma Gandhi</p>
<p>Hate cannot drive out hate, only love can do that. - Martin Luther King, Jr.</p>
```

### Implicit transitions

The default language in a code block is C#, but the Razor Page can transition back to HTML:

```cshtml
@{
    var inCSharp = true;
    <p>Now in HTML, was in C# @inCSharp</p>
}
```

### Explicit delimited transition

To define a subsection of a code block that should render HTML, surround the characters for rendering with the Razor **\<text>** tag:

```cshtml
@for (var i = 0; i < people.Length; i++)
{
    var person = people[i];
    <text>Name: @person.Name</text>
}
```

Use this approach to render HTML that isn't surrounded by an HTML tag. Without an HTML or Razor tag, a Razor runtime error occurs.

The **\<text>** tag is useful to control whitespace when rendering content:

* Only the content between the **\<text>** tag is rendered. 
* No whitespace before or after the **\<text>** tag appears in the HTML output.

### Explicit Line Transition with @:

To render the rest of an entire line as HTML inside a code block, use the `@:` syntax:

```cshtml
@for (var i = 0; i < people.Length; i++)
{
    var person = people[i];
    @:Name: @person.Name
}
```

Without the `@:` in the code, a Razor runtime error is generated.

Warning: Extra `@` characters in a Razor file can cause compiler errors at statements later in the block. These compiler errors can be difficult to understand because the actual error occurs before the reported error. This error is common after combining multiple implicit/explicit expressions into a single code block.

## Control Structures

Control structures are an extension of code blocks. All aspects of code blocks (transitioning to markup, inline C#) also apply to the following structures:

### Conditionals @if, else if, else, and @switch

`@if` controls when code runs:

```cshtml
@if (value % 2 == 0)
{
    <p>The value was even.</p>
}
```

`else` and `else if` don't require the `@` symbol:

```cshtml
@if (value % 2 == 0)
{
    <p>The value was even.</p>
}
else if (value >= 1337)
{
    <p>The value is large.</p>
}
else
{
    <p>The value is odd and small.</p>
}
```

The following markup shows how to use a switch statement:

```cshtml
@switch (value)
{
    case 1:
        <p>The value is 1!</p>
        break;
    case 1337:
        <p>Your number is 1337!</p>
        break;
    default:
        <p>Your number wasn't 1 or 1337.</p>
        break;
}
```

### Looping @for, @foreach, @while, and @do while

Templated HTML can be rendered with looping control statements. To render a list of people:

```cshtml
@{
    var people = new Person[]
    {
          new Person("Weston", 33),
          new Person("Johnathon", 41),
          ...
    };
}
```

The following looping statements are supported:

`@for`

```cshtml
@for (var i = 0; i < people.Length; i++)
{
    var person = people[i];
    <p>Name: @person.Name</p>
    <p>Age: @person.Age</p>
}
```

`@foreach`

```cshtml
@foreach (var person in people)
{
    <p>Name: @person.Name</p>
    <p>Age: @person.Age</p>
}
```

`@while`

```cshtml
@{ var i = 0; }
@while (i < people.Length)
{
    var person = people[i];
    <p>Name: @person.Name</p>
    <p>Age: @person.Age</p>

    i++;
}
```

`@do while`

```cshtml
@{ var i = 0; }
@do
{
    var person = people[i];
    <p>Name: @person.Name</p>
    <p>Age: @person.Age</p>

    i++;
} while (i < people.Length);
```

### Compound @using

In C#, a `using` statement is used to ensure an object is disposed. In Razor, the same mechanism is used to create HTML Helpers that contain additional content. In the following code, HTML Helpers render a form tag with the `@using` statement:


```cshtml
@using (Html.BeginForm())
{
    <div>
        email:
        <input type="email" id="Email" value="">
        <button>Register</button>
    </div>
}
```

Scope-level actions can be performed with [Tag Helpers](xref:mvc/views/tag-helpers/intro).

### @try, catch, finally

Exception handling is similar to C#:

[!code-cshtml[Main](razor/sample/Views/Home/Contact7.cshtml)]

### @lock

Razor has the capability to protect critical sections with lock statements:

```cshtml
@lock (SomeLock)
{
    // Do critical section work
}
```

### Comments

Razor supports C# and HTML comments:

```cshtml
@{
    /* C# comment */
    // Another C# comment
}
<!-- HTML comment -->
```

The code renders the following HTML:

```html
<!-- HTML comment -->
```

Razor comments are removed by the server before the webpage is rendered. Razor uses `@*  *@` to delimit comments. The following code is commented out, so the server doesn't render any markup:

```cshtml
@*
    @{
        /* C# comment */
        // Another C# comment
    }
    <!-- HTML comment -->
*@
```

## Directives

Razor directives are represented by implicit expressions with reserved keywords following the `@` symbol. A directive typically changes the way a view is parsed or enables different functionality.

Understanding how Razor generates code for a view makes it easier to understand how directives work.

[!code-html[Main](razor/sample/Views/Home/Contact8.cshtml)]

The code generates a class similar to the following:

```csharp
public class _Views_Something_cshtml : RazorPage<dynamic>
{
    public override async Task ExecuteAsync()
    {
        var output = "Getting old ain't for wimps! - Anonymous";

        WriteLiteral("/r/n<div>Quote of the Day: ");
        Write(output);
        WriteLiteral("</div>");
    }
}
```

Later in this article, the section [Viewing the Razor C# class generated for a view](#viewing-the-razor-c-class-generated-for-a-view) explains how to view this generated class.

### @using

The `@using` directive adds the C# `using` directive to the generated view:

[!code-cshtml[Main](razor/sample/Views/Home/Contact9.cshtml)]

### @model

The `@model` directive specifies the type of the model passed to a view:

```cshtml
@model TypeNameOfModel
```

In an ASP.NET Core MVC app created with individual user accounts, the *Views/Account/Login.cshtml* view contains the following model declaration:

```cshtml
@model LoginViewModel
```

The class generated inherits from `RazorPage<dynamic>`:

```csharp
public class _Views_Account_Login_cshtml : RazorPage<LoginViewModel>
```

Razor exposes a `Model` property for accessing the model passed to the view:

```cshtml
<div>The Login Email: @Model.Email</div>
```

The `@model` directive specifies the type of this property. The directive specifies the `T` in `RazorPage<T>` that the generated class that the view derives from. If the `@model` directive isn't specified, the `Model` property is of type `dynamic`. The value of the model is passed from the controller to the view. For more information, see [Strongly typed models and the @model keyword.

### @inherits

The `@inherits` directive provides full control of the class the view inherits:

```cshtml
@inherits TypeNameOfClassToInheritFrom
```

The following code is a custom Razor page type:

[!code-csharp[Main](razor/sample/Classes/CustomRazorPage.cs)]

The `CustomText` is displayed in a view:

[!code-cshtml[Main](razor/sample/Views/Home/Contact10.cshtml)]

The code renders the following HTML:

```html
<div>Custom text: Gardyloo! - A Scottish warning yelled from a window before dumping a slop bucket on the street below.</div>
```

 `@model` and `@inherits` can be used in the same view. `@inherits` can be in a *_ViewImports.cshtml* file that the view imports:

[!code-cshtml[Main](razor/sample/Views/_ViewImportsModel.cshtml)]

The following code is an example of a strongly-typed view:

[!code-cshtml[Main](razor/sample/Views/Home/Login1.cshtml)]

If "rick@contoso.com" is passed in the model, the view generates the following HTML markup:

```html
<div>The Login Email: rick@contoso.com</div>
<div>Custom text: Gardyloo! - A Scottish warning yelled from a window before dumping a slop bucket on the street below.</div>
```

### @inject


The `@inject` directive enables the Razor Page to inject a service from the [service container](xref:fundamentals/dependency-injection) into a view. For more information, see [Dependency injection into views](xref:mvc/views/dependency-injection).

### @functions

The `@functions` directive enables a Razor Page to add function-level content to a view:

```cshtml
@functions { // C# Code }
```

For example:

[!code-cshtml[Main](razor/sample/Views/Home/Contact6.cshtml)]

The code generates the following HTML markup:

```html
<div>From method: Hello</div>
```

The following code is the generated Razor C# class:

[!code-csharp[Main](razor/sample/Classes/Views_Home_Test_cshtml.cs?range=1-19)]

### @section

The `@section` directive is used in conjunction with the [layout](xref:mvc/views/layout) to enable views to render content in different parts of the HTML page. For more information, see [Sections](xref:mvc/views/layout#layout-sections-label).

## Tag Helpers

There are three directives that pertain to [Tag Helpers](xref:mvc/views/tag-helpers/intro).

| Directive | Function |
| --------- | -------- |
| [@addTagHelper](xref:mvc/views/tag-helpers/intro#add-helper-label) | Makes Tag Helpers available to a view. |
| [@removeTagHelper](xref:mvc/views/tag-helpers/intro#remove-razor-directives-label) | Removes Tag Helpers previously added from a view. |
| [@tagHelperPrefix](xref:mvc/views/tag-helpers/intro#prefix-razor-directives-label) | Specifies a tag prefix to enable Tag Helper support and to make Tag Helper usage explicit. |

## Razor reserved keywords

### Razor keywords

* page (Requires ASP.NET Core 2.0 and later)
* functions
* inherits
* model
* section
* helper (Not currently supported by ASP.NET Core)

Razor keywords are escaped with `@(Razor Keyword)` (for example, `@(functions)`).

### C# Razor keywords

* case
* do
* default
* for
* foreach
* if
* else
* lock
* switch
* try
* catch
* finally
* using
* while

C# Razor keywords must be double-escaped with `@(@C# Razor Keyword)` (for example, `@(@case)`). The first `@` escapes the Razor parser. The second `@` escapes the C# parser.

### Reserved keywords not used by Razor

* namespace
* class

## Viewing the Razor C# class generated for a view

Add the following class to the ASP.NET Core MVC project:

[!code-csharp[Main](razor/sample/Utilities/CustomTemplateEngine.cs)]

Override the `RazorTemplateEngine` added by MVC with the `CustomTemplateEngine` class:

[!code-csharp[Main](razor/sample/Startup.cs?highlight=4&range=10-14)]

Set a break point on the `return csharpDocument` statement of `CustomTemplateEngine`. When program execution stops at the break point, view the value of `generatedCode`.

![Text Visualizer view of generatedCode](razor/_static/tvr.png)

## View lookups and case sensitivity

The Razor view engine performs case-sensitive lookups for views. However, the actual lookup is determined by the underlying file system:

* File based source: 
  * On operating systems with case insensitive file systems (for example, Windows), physical file provider lookups are case insensitive. For example, `return View("Test")` results in matches for */Views/Home/Test.cshtml*, */Views/home/test.cshtml*, and any other casing variant.
  * On case-sensitive file systems (for example, Linux, OSX, and with `EmbeddedFileProvider`), lookups are case-sensitive. For example, `return View("Test")` specifically matches */Views/Home/Test.cshtml*.
* Precompiled views: With ASP.NET Core 2.0 and later, looking up precompiled views is case insensitive on all operating systems. The behavior is identical to physical file provider's behavior on Windows. If two precompiled views differ only in case, the result of lookup is non-deterministic.

Developers are encouraged to match the casing of file and directory names to the casing of:

    * Area, controller, and action names. 
    * Razor Pages.
    
Matching case ensures the deployments find their views regardless of the underlying file system.
