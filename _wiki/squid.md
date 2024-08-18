---
title: Squid
---

### Access Control

Squid 的 access control 机制包括 ACL elements 和 Access lists 两个部分。

#### ACL elements

格式如下：

	acl aclname acltype value1 [OR] value2

(OR 表示暗含的逻辑，不是语法的一部分）

比如：

	acl localnet src 10.0.0.0/8     # RFC1918 possible internal network
	acl localnet src 172.16.0.0/12  # RFC1918 possible internal network
	acl localnet src 192.168.0.0/16 # RFC1918 possible internal network

不同类型的 acl elements 不可取同一个 aclname。
可以为同一个 aclname 分多行指定 value，这些多行的 value 将被组合成一个 list（OR 逻辑）。

#### Access Lists[1]

一个 access list *rule* 包含一个 allow 或 deny 关键字，后跟一个或多个 acl element 名字。
一个 access list 包含一个或多个 access list rules，分布在多行。

这些 rules 会按照书写的顺序排队检查，如果一旦找到一个匹配，那么检查将终止。

如果一个 rule 有多个 acl elements，那么这些 acl elements 按照 AND 逻辑组合起来。

总体来说，access list 的处理逻辑如下：

	http_access allow|deny acl AND acl AND ...
        OR
	http_access allow|deny acl AND acl AND ...
        OR
	...	

如果检查下来没有任何一项 rule 匹配，那么采取的默认动作是书写的最后一条 rule 的相反动作。因此最好写一条包含 `all` element 的 rule 来兜底。

	http_access deny all

其中 `all` 为 squid 预定义的一条 element：

	acl all src all


### 缓存

#### refresh-pattern[2]

refresh_pattern的作用: 用于确定一个页面进入cache后，它在cache中停留的时间。

	usage: refresh_pattern [-i] regex min percent max [options]

min: 如果一个对象没有明确的过期时间，那么停留在 cache 中在此时间内将它看作 fresh；
max：如果一个对象没有明确的过期时间，那么停留在 cache 中此时间以后将它看作 stale；
percent: 如果请求的对象在 cache 中停留的时间正好处在这两个时间之间呢？它可能过期也可能不过期。这个时候就需要用一个叫做 LM-factor 的算式来计算了。首先明确几个概念：

	resource_age =对象进入cache的时间 - 对象的last_modified
	response_age =当前时间 - 对象进入cache的时间
	LM-factor=(response_age) / (resource_age)

假设一个对象的 last_modified 时间是 `2015/07/09 13:30`，进入 cache 的时间是 `2015/07/09 14:30`，那么 resource_age 等于 60 分钟。如果请求这个对象的时间是 `2015/07/09 14:40`，那么 response_age 等于 10 分钟。因此 LM-factor = 10/60 = 17%。

这个时候就很好判断了：

	LM-factor < percent, fresh
	LM-factor > percent, stale

当一个请求对象过期后，再次请求它是 squid 会跟上游确认对象是否有改变。如果有改变，拉取到 cache 中。如果没有改变，cache 中的对象不变，只是将其进入 cache 的时间设定为上游响应时间，这样一来 resource_age 就会变大，相应地对象在 cache 中存活的时间也就越长。

总体来说 squid 收到一个页面请求时：

1. 计算出 response_age，
2. 如果 response_age < min 则 fresh；如果response_age > max 则 stale
3. 如果response_age在之间，如果response_age < 存活时间，fresh，否则stale

[1]: http://wiki.squid-cache.org/SquidFaq/SquidAcl)
[2]: http://www.squid-cache.org/Versions/v3/3.4/cfgman/refresh_pattern.html  