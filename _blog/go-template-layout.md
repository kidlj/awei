---
title: Go Template Layout
date: 2021-09-15
---

Go 1.16 引入了 embed package，可以将非 .go 文件打包到二进制文件中，极大地方便了 Go 程序的部署。标准库中 html/template 也同步增加了 `ParseFS` 函数，用于将 embed.FS 内包含的所有模板文件一整个编译成一个 template tree。

File: templates.go

    package templates

    import (
        "embed"
        "html/template"
    )

    //go:embed views/*.html
    var tmplFS embed.FS

    type Template struct {
        templates *template.Template
    }

    func New() *Template {
        funcMap := template.FuncMap{
            "inc": inc,
        }

        templates := template.Must(template.New("").Funcs(funcMap).ParseFS(tmplFS, "views/*.html"))
        return &Template{
            templates: templates,
        }
    }


File: main.go

    t := templates.New()

`t.templates` 是一个包含了所有匹配 `views/*.html` 模板文件的一个全局 template，所有这些模板互相关联可以相互引用，模板的名字就是文件的名字，比如 `article.html`。

进一步，我们给 `*Template` 类型定义一个 `Render` 方法，用于实现 Echo web 框架的 `Renderer` 接口。

    // templates.go
    func (t *Template) Render(w io.Writer, name string, data interface{}, c echo.Context) error {
        return t.templates.ExecuteTemplate(w, name, data)
    }

然后就可以为 Echo 指定 renderer，方便在每个 handler 生成 HTML 响应，只需要向 `c.Render` 函数传递模板的名字即可。

File: main.go

    func main() {
        t := templates.New()

        e := echo.New()
        e.Renderer = t
    }


File: handler.go

    func (h *Handler) articlePage(c echo.Context) error {
        id := c.Param("id")
        article, err := h.service.GetArticle(c.Request().Context(), id)
        ...
        return c.Render(http.StatusOK, "article.html", article)
    }

因为 `t.templates` 模板包含了所有的模板文件，因此每一个模板名字都可以直接使用。

为了实现 HTML 的组装，我们需要用到模板的继承。比如定义一个 layout.html 用于基本的 HTML 框架和 `<head>` 元素，并且设定 {% raw %}`{{block "title"}}`{% endraw %} 和 {% raw %}`{{block "content"}}`{% endraw %}，其它模板继承 layout.html，并且用自己定义的 blocks 填充或覆盖 layout 模板的同名 blocks。

以下是 layout.html 模板的内容：

    <!DOCTYPE html>
    <html lang="en">

    <head>
        <meta charset="UTF-8">
        <meta http-equiv="X-UA-Compatible" content="IE=edge">
        <meta name="viewport" content="width=device-width, initial-scale=1.0">
        <title>{{block "title" .}}{{end}}</title>
        <script src="/static/main.js"></script>
    </head>

    <body>
        <div class="main">{{block "content" .}}{{end}}</div>
    </body>

    </html>

其它的模板，可以引用（继承）layout.html，并定义 layout.html 模板中的 blocks。

比如 login.html 内容如下：

    {% raw %}
    {{template "layout.html" .}}

    {{define "title"}}登录{{end}}

    {{define "content"}}
    <form class="account-form" method="post" action="/account/login" data-controller="login">
        <div div="account-form-title">登录</div>
        <input type="phone" name="phone" maxlength="13" class="account-form-input" placeholder="手机号" tabindex="1">
        <div class="account-form-field-submit ">
            <button type="submit" class="btn btn-phone">登录</button>
        </div>
    </form>
    {{end}}
    {% endraw %}

article.html 同样也引用 layout.html:

    {% raw %}
    {{template "layout.html" .}}

    {{define "title"}}<h1>{{.Title}}</h1>{{end}}

    {{define "content"}}
    <p>{{.URL}}</p>
    <article>{{.Content}}</article>
    {{end}}
    {% endraw %}

我们期望在渲染 login.html 模板的时候，它定义的 blocks 覆盖 layout.html 的 blocks，在渲染 article.html 模板的时候同样如此。可事实不是这样，这归咎于 Go text/template 的实现。在我们执行 `ParseFS(tmplFS, "views/*.html")` 的过程中，假设 article.html 首先被解析，它其中的 `content` block 也被解析成一个模板名，那么后续再解析 login.html 模板时，在其中又发现了 `content` block，text/template 就会用后边解析的内容覆盖同名的模板，所以等所有模板解析完成，实际上我们模板树中只存在一个名为 `content` 的模板，就是最后被解析的那一个模板文件内定义的 `content`。

因此，在我们执行 article.html 模板的时候，可能其中的 `content` 模板并不是这个模板内定义的内容，而是其它模板内定义的 `content` 内容。

针对这个问题，社区里提出了一些解决方案。比如不使用全局模板，每次渲染时创建一个新模板，只包含 layout.html 和子模板的内容。可这样做实在繁琐。实际上，在 Go 1.6 版本为 text/template 引入 `block` 指令[github]的时候，我们可以配合 `Clone` 方法实现我们想要的功能，只需要对上边的代码做一点更改。

File: templates.go

    package templates

    import (
        "embed"
        "html/template"
        "io"

        "github.com/labstack/echo/v4"
    )

    //go:embed views/*.html
    var tmplFS embed.FS

    type Template struct {
        templates *template.Template
    }

    func New() *Template {
        funcMap := template.FuncMap{
            "inc": inc,
        }

        templates := template.Must(template.New("").Funcs(funcMap).ParseFS(tmplFS, "views/*.html"))
        return &Template{
            templates: templates,
        }
    }

    func (t *Template) Render(w io.Writer, name string, data interface{}, c echo.Context) error {
        tmpl := template.Must(t.templates.Clone())
        tmpl = template.Must(tmpl.ParseFS(tmplFS, "views/"+name))
        return tmpl.ExecuteTemplate(w, name, data)
    }

可以看到这里只修改了 `Render` 函数。之前我们用全局模板执行其中的一个引用了 layout.html 的子模板，会导致同名 block 定义内容的错乱，现在我们不直接执行这个全局模板，而是先将它克隆成一个新模板，这个新模板里的 `content` block 可能也不是我们想要的，所以这里在这个模板之上再解析一个我们最终要渲染的子模板的内容，这样新添加的子模板的 `content` 内容会覆盖之前的可能错误的 `content`。我们的目标子模板里引用了全局模板中的 layout.html，而 layout.html 是没有重名的，而且因为全局模板从来没有被执行过（每次执行我们都在 `Render` 函数里克隆出一个新全局模板），所以它也是干净的。最终执行某个模板的时候，我们有了一个干净的 layout.html，以及我们想要的 `content` 内容，这就相当于我们每次执行时都生成一个新模板，这个模板只包含我们需要的 layout 模板和子模板。思路是一样的，只是这里不需要执行模板时手动生成新模板，而是在 `Render` 函数里自动完成了。

当然，你也可以在子模板里使用 {% raw %}`{{ template }}`{% endraw %} 引用别的 layout 模板，只要这些 layout 模板没有重名就不会互相覆盖，在执行时候只需要指定目标子模板的名字，模板引擎会自动根据其中定义的 {% raw %}`{{ template }}`{% endraw %} tag 为我们寻找 layout 模板，这些模板都在克隆出的全局模板里了。

[github]: https://github.com/golang/go/commit/12dfc3bee482f16263ce4673a0cce399127e2a0d