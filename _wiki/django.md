---
title: Django
---




### Models

示例代码：

	from django.db import models

	class Blog(modes.Model):
		name = models.CharField(max_length=100)
		tagline = models.TextField()

		def __str__(self):
			return self.name

	class Author(models.Model):
		name = models.CharField(max_length=50)
		email = models.EmailField()

		def __str__(self):
			return self.name

	class Entry(models.Model):
		blog = models.ForeignKey(Blog)
		headline = models.CharField(max_length=255)
		body_text = models.TextField()
		pub_date = models.DateField()
		mod_date = models.DateField()
		authors = models.ManyToManyField(Author)
		n_comments = models.IntegerField()
		n_pingbacks = models.IntegerField()
		rating = models.IntegerField()

		def __str__(self):
			return self.headline


#### 原理

模型中的 Class 对应的是数据库中的「表」;  
Class 中的 Field 对应的是数据库表中的「字段」;  
一个类实例对应的是数据库表中的一条「记录」。

#### Saving changes to objects

- Saving value fields:

		>>> b = Blog(name='IV', tagline='All about IV')
		>>> b.save()

- Saving ForeignKey fields:

		>>> entry = Entry.objects.get(pk=1)
		>>> iv_blog = Blog.objects.get(name='IV')
		>>> entry.blog = iv_blog
		>>> entry.save()

- Saving ManyToManyField fields:

		>>> joe = Author.objects.create(name='Joe')
		>>> entry.authors.add(joe)		# joe is an 'object' created

#### Retrieving Objects

从数据库获取对象，要通过模型提供的 Manager 来构建 QuerySet.

一个 QuerySet 代表数据库中的一组对象(记录)集合。可以在 QuerySet 之上使用一个或多个 filters。用 SQL 术语来说，QeurySet 相当于 `SELECT` 语句，filter 则相当于 `WHERE` 或者 `LIMIT`。

通过模型提供的 Manager 来构建 QuerySet。每个模型都有至少一个 Manager，默认的一个叫做 objects。以下可以获取数据表中的所有实例：

	>>> Entry.objects.all()

为了反映获取 QuerySet 是一个数据表级别而不是记录级别的操作这个事实，Manager 只能用类来访问，不能使用类实例。

	>>> Blog.objects	# good
	>>> b = Blog(name='Foo', tagline='Bar')
	>>> b.objects		# bad

通过对一个 QuerySet 应用 `filter` 或者 `exclude` 会返回另外一个 QuerySet，因此可以将它们串起来：

	>>> Entry.objects.all().filter(
	...		headline__startswith='What'
	... ).exclue(
	...		pub_date__gte=datetiem.date.today()
	... ).filter(
	...	    pub_date__gte=datetime(2005, 1, 30)
	... )

因为 `filter()` 只会返回 QuerySet，即使只获取到一个对象的时候。如果你确定只有一个实例返回，那么可在 Manager 上使用 `get()` 方法：

	>>> one_entry = Entry.objects.get(pk=1)

如果获取失败，则抛出 `Entry.DoesNotExist` 异常，如果有多个获取，抛出 `MultipleObjectsReturned` 异常。

既然 QuerySet 是一个列表，所以它支持分片获取：

	>>> Entry.objects.order_by('headline')[0]

#### Field Lookups

字段查询相当于 SQL 中的 `WHERE` 语句。通过给 `filter()` 等方法上加上关键字参数来应用于 QuerySet 之上。关键字参数的通用格式为：

	field__lookuptype=value

比如：
	
	>>> Entry.objects.filter(pub_date__lte='2015-01-21')

#### Field Lookups that span relationships

Django 支持跟随数据模型定义的关系来获取对象，这背后是对 SQL `JOIN` 语句的自动处理。而且关系跟随可以有任意深度。

- 正向查询，使用小写的 model 和 field 名，中间用 `--` 隔开：

		>>> Entry.objects.filter(blog__name='Beatles Blog')

- 反向查询，使用小写的 model 名和 field 名：

		>>> Blog.objects.filter(entry__headline__contains='Lennon')

要始终区分，我们 filter 的是什么，以下这两者是不等同的：

	>>> Blog.objects.filter(entry__headline__contains='Lennon',
	...			entry__pub_date__year=2008)

	>>> Blog.objects.filter(entry__headline__contains='Lennon).filter(
	...			entry__pub_date__year=2008)

第一个查询的是这样的 Blog 对象：链接到该对象的某一个 Entry 对象里其 headline 既包含 'Lennon'，pub_date 年份又是 2008；

第二个查询的是这样的 Blog 对象：链接到该对象的某一个 Entry 对象的 headline 包含 'Lenno'，又有一个链接到它的 Entry 对象（可以跟前边的是同一个）的 pub_date 年份是 2008。


#### ForeignKey Related Objects

如果一个模型定义了和另一个模型的关系，比如 ForeignKey, ManyToManyField 和 OneToOneFiled，那么通过其中一个模型的实例对象，可以方便的访问与其相关联的另一模型的实例对象。


* 正向，通过 field 名：

		>>> e = Entry.objects.get(id=2)
		>>> e.blog 		# 返回相关对象

* 反向，通过 Manager

	默认情况下，Manager 名为 `FOO_set`，这里 `FOO` 是关系源模型的名字。

		>>> b = Blog.objects.get(id=1)
		>>> b.entry_set.all()
		# b.entry_set 是一个返回 QuerySet 的 Manager
		>>> b.entry_set.filter(headline__contains='Lennon')
		>>> b.entry_set.count()

	可以通过在关系源模型里设置 `related_name` 来覆盖 `FOO_set`:

		blog = ForeignKey(Blog, related_name='entries')

	这样一来：

		>>> b = Blog.objects.get(id=1)
		>>> b.entries.all()

	除了以上使用的 QuerySet 方法外，ForeignKey Manager 还定义了更多方法：

	*  `add(obj1, obj2, ...)`
	*  `create(**kwargs)`
	*  `remove(obj1, obj2, ...)`
	*  `clear()`

	还可以这样使用：

		>>> b = Blog.objects.get(id=1)
		>>> b.entry_set = [e1, e2] 		# e1, e2 是 Entry 实例，须先创建


#### ManyToManyField Related Objects

同 ForeignKey 关系一样，Django 里的 ManyToManyField 关系也只需要在一方模型里定义。但是通过一方对象实例访问另一方对象实例的能力却是双向的。

- 正向：使用 field 名 Manager 
- 反向：使用名为 `FOO_set` 的 Manager

举例：

	>>> e = Entry.objects.get(id=1)
	>>> e.authors.all()
	>>> e.authors.count()
	>>> e.authors.filter(name__contains='John')

	>>> a = Author.objects.get(id=5)
	>>> a.entry_set.all()

#### Using A Custom Reverse Manager

示例代码：

	from django.db import models

	class Entry(models.Model):
		objects = models.Manager()	# Default Manager
		entries = EntryManager()	# Custom Manager

如此一来：

	>>> b = Blog.objects.get(id=1)
	>>> b.entry_set(manager='entries').all()
	>>> b.entry_set(manager='entries').is_published()	# custom method
	
### Views

#### django.conf.urls.url

`url` 函数接受四个参数，分别是 regex, view, kwargs 和 name.

当找到一个正则匹配时，就调用后面的 view 函数，将一个 `HttpRequest` 对象作为第一个参数，将正则表达式捕获的内容作为第二个参数。

如果正则表达式采用的是简单捕获，则传递的是位置参数；如果用的命名捕获，则传递的是关键字参数。


#### Namespace

从 polls/views.py 中加载 'polls/detail.html'，这里用到了命名空间，实现方式是在建立如下的目录层级：

	BASE_DIR/polls/templates/polls/detail.html

同样地，从 polls/index.html 中用 `url polls:detail` 指定 URL 也用到了命名空间。



### Templates

#### Custom Template Tags

我给 Wagtail 的一个页面写了如下的一个 custom template tag：

	@register.simple_tag
	def related_page(title, calling_page):
		try:
			related_item = calling_page.related_pages.get(title=title)
			return related_item.url
		except:
			return "#"

可是当这个 `related_item` 不存在的时候，template 仍然会报出 `DoesNotExist` 异常。因此我猜想在 custom template tag 的函数里不能使用异常捕获。

Django 文档里有这么一段[doc]：

> Usually any exception raised from a template filter will be exposed as a server error. Thus, filter functions should avoid raising exceptions if there is a reasonable fallback value to return. In case of input that represents a clear bug in a template, raising an exception may still be better than silent failure which hides the bug.

StackOverflow 上也有一个回答里说到[stackoverflow]：

> Your template should not be raising an exception as a normal course of action. If there's an error in the template, you fix it. Otherwise, anything that could potentially raise an exception should be handled in the model or the view. There's no tag like you mention for a reason.


#### Accessing method calls

可以在 template 中调用 contex 中的对象定义的方法。比如，对处于 ForeignKey 关系中的对象，可以使用 QuerySets 自动提供的方法：

	{% raw %}
	{% for comment in task.comment_set.all %}
		{{ comment }}
	{% endfor %}
	{% endraw %}

以及，

	{% raw %}
	{{ task.comment_set.all.count }}
	{% endraw %}

当然，也可以使用在 model 里自定义的方法：

	class Task(models.Model):
		def foo(self):
			return "bar"

在 template 里：

	{% raw %}
	{{ task.foo }}
	{% endraw %}


_注意_: 因为 Django 故意对 template 的逻辑处理能力做了限制，因此在 template 里使用的对象方法不能添加任何参数。数据应该在 view 里准备好，然后让 template 来呈现。

[doc]: https://docs.djangoproject.com/en/1.7/howto/custom-template-tags/
[stackoverflow]: http://stackoverflow.com/questions/8524077/catching-exceptions-in-django-templates
