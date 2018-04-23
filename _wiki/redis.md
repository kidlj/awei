---
title: Redis
---

### 检查 db 大小

1. 先找出 bigkeys:

    $ redis-cli -n 20 --bigkeys

对于 string 类型，会给出 key 的数目和平均大小（字节）。

对于 set 类型，会给出 member 比较多的 key 和总的 member 数。

2. 判断类型为 set 的 db 占用内存数

    $ redis-cli -n 20 DEBUG OBJECT <keyName> 

找到输出里的 `serializedlength:1175841791`，这里是整个 set 的字节数。用它除以 set 里 member 的数量，就是每个 member 占用的字节数。

再根据每个 member 占用的字节数乘以 `--bigkeys` 得出的所有 member 的数目就是 db 占用的总共字节数（近似）。
