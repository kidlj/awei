---
title: Go HTML Template Layout - EN
---

Go 1.16 introduces the embed package, which packages non-.go files into binaries, greatly facilitating the deployment of Go programs. The `ParseFS` function was also added to the standard library html/template, which compiles all the template files contained in embed.FS into a template tree.

```go
// templates.go
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


// main.go
t := templates.New()
```

`t.templates` is a global template that contains all matching `views/*.html` templates, all of which are related and can be referenced to each other, and the name of the template is the name of the file, e.g. `article.html`.

Further, we define a `Render` method for the `*Template` type, which implements the `Renderer` interface of the Echo web framework.

```go
// templates.go
func (t *Template) Render(w io.Writer, name string, data interface{}, c echo.Context) error {
	return t.templates.ExecuteTemplate(w, name, data)
}
```

You can then specify the renderer for Echo to facilitate generating HTML responses at each handler, simply by passing the name of the template to the `c.Render` function.

```go
// main.go
func main() {
	t := templates.New()

	e := echo.New()
	e.Renderer = t
}


// handler.go
func (h *Handler) articlePage(c echo.Context) error {
	id := c.Param("id")
	article, err := h.service.GetArticle(c.Request().Context(), id)
	...
	return c.Render(http.StatusOK, "article.html", article)
}
```

Since the `t.templates` template contains all the parsed templates, each template name can be used directly.

In order to assemble HTMLs, we need to use template inheritance. For example, define a layout.html for the basic HTML frame and the `<head>` element, and set {% raw %}`{{block "title"}}`{% endraw %} and {% raw %}`{{block "content"}}`{% endraw %}, other templates inherit layout.html, and populate or override the layout template's blocks of the same name with their own defined blocks.

The following is the content of the layout.html template.

{% raw %}
```html
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
```
{% endraw %}

For other templates, you can refer to (inherit from) layout.html and define the blocks in the layout.html template.

For example, login.html reads as follows.

{% raw %}
```html
{{template "layout.html" .}}

{{define "title"}}Login{{end}}

{{define "content"}}
<form class="account-form" method="post" action="/account/login" data-controller="login">
	<div div="account-form-title">Login</div>
	<input type="phone" name="phone" maxlength="13" class="account-form-input" placeholder="Phone" tabindex="1">
	<div class="account-form-field-submit ">
		<button type="submit" class="btn btn-phone">Login</button>
	</div>
</form>
{{end}}
```
{% endraw %}

article.html also references layout.html:

{% raw %}
```html
{{template "layout.html" .}}

{{define "title"}}<h1>{{.Title}}</h1>{{end}}

{{define "content"}}
<p>{{.URL}}</p>
<article>{{.Content}}</article>
{{end}}
```
{% endraw %}

We would expect the blocks defined in the login.html template to override the blocks in layout.html when rendering it, and also when rendering the article.html template. But that's not the case, and it's down to the Go text/template implementation. In our implementation of `ParseFS(tmplFS, "views/*.html")`, suppose article.html is parsed first and its `content` block is parsed as a template name, then when the login.html template is parsed later and a`content` block is also found in it, text/template will overwrite the template of the same name with the later parsed content, so when all the templates are parsed, there is actually only one template named `content` in our template tree, which is the `content` defined in the last parsed template file.

Therefore, when we execute the article.html template, it is possible that the `content` template is not the content defined in this template, but the `content` defined in other templates.

The community has proposed some solutions to this problem. For example, instead of using a global template, a new template is created each time it is rendered, containing only the contents of layout.html and the sub-template. But this is really tedious. In fact, when Go 1.6 introduced the `block` directive[1] for text/template, we were able to do what we wanted with the `Clone` method, with just a few changes to the code above.

```go
// templates.go
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
```

You can see that only the `Render` function has been modified here. Instead of executing the global template, we will clone it into a new template, and the `content` block in this new template may not be the one we want, so here we parse the content of a sub-template we will eventually render on top of this global template, so that the `content` of the newly added sub-template will overwrite the previous, possibly incorrect `content`. Our target sub-template references layout.html in the global template, which is not conflicted, and since the global template is never executed (we clone a new global template in the `Render` function each time it is executed), it is also clean. When a template is finally executed, we have a clean layout.html with the `content` content we want, which is equivalent to generating a new template each time we execute it, which contains only the layout template and sub-templates we need. The idea is the same, but instead of manually generating a new template when executing the template, it's done automatically in the `Render` function.

Of course, you can also use {% raw %}`{{ template }}`{% endraw %} to refer to other layout templates in the sub-template, as long as these layout templates do not overwrite each other, you just need to specify the name of the target sub-template when executing, and the template engine will automatically use the {% raw %}`{{ template }}`{% endraw %} tag defined in it to find the layout templates for us, which are all in the cloned global template.

[1]: https://github.com/golang/go/commit/12dfc3bee482f16263ce4673a0cce399127e2a0d