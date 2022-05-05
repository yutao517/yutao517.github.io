---
layout: article
title: Linux修改DNS解析加速国内使用github
tags: Linux
category: blog
date: 2022-05-05 13:40:00 +08:00
mermaid: true
---

```bash
vim /etc/hosts
```

```
52.69.186.44 github.com
52.192.72.89 github.com
140.82.114.4    github.com
```
```
185.199.110.133 raw.githubusercontent.com
185.199.109.133 raw.githubusercontent.com
185.199.108.133 raw.githubusercontent.com
185.199.111.133 raw.githubusercontent.com
```
这几行解析地址可以任选两个，目前都比较快。

可以看到已经ping通

![在这里插入图片描述](https://img-blog.csdnimg.cn/8242faf39eaf4a4e979fe177678aa98a.png)

如果后期无法使用，访问站长工具

[链接](https://tool.chinaz.com/speedworld/)

搜索github.com，raw.githubusercontent.com

![在这里插入图片描述](https://img-blog.csdnimg.cn/aa5cd6ece0fa43568752bf7ee86ab82f.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/62fa9de2bbd44cdc8315d28b886fdd4f.png)

下滑更改为这些解析地址

![在这里插入图片描述](https://img-blog.csdnimg.cn/73bd0093917a454d973c93ac1b3d9306.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/b01f2f4215a747ccbfbaed04ec631e8d.png)

