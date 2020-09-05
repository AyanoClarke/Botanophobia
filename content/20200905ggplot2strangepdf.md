+++

title = "ggplot2 辣眼睛输出一则"
description = "用R绘制图片并没有想象的那么简单，处处藏着坑"
date=2020-09-05

[taxonomies]
categories = ["这是什么"]
tags = ["R"]

[extra]
author = "Clarke"
relative_posts=[]

+++

最近遇到一个辣眼睛的情况，直接放绘制的代码：

```R
library(ggplot2)
library(extrafont)

p <- ggplot() +
  annotate("text",
    x = 0, y = 0,
    label = "italic(PR123)-like", parse = TRUE,
    family = "Times New Roman"
  ) 

ggsave("~/Desktop/test.pdf", p, height = 2)
```

![辣眼睛的第一版](https://i.loli.net/2020/09/05/COVscIgwtxUnJDN.png)

诶，是不是觉得自己的眼睛有点模糊，为什么 `123` 的`2`italic了，而`1`和`3`却没有，为什么 `P` 那么粗？怎么办怎么办？
定睛一看，似乎忘记添加引号了。

```R
ggplot() +
  annotate("text",
    x = 0, y = 0,
    label = "italic(\"PR123\")-like", parse = TRUE,
    family = "Times New Roman"
  ) 
```

![辣眼睛的第二版](https://i.loli.net/2020/09/05/jmfq7tUoacwOZnA.png)
输出结果看上去把italic的事情解决了，但是似乎的问题解决了，但是 `PR123` 总觉得比正常的 Times New Roman 的 italic 更加粗，更加倾斜。这时候突然发现，似乎没有那么可怕，
如果不用 Times New Roman，而是用默认的 Helvetica，就没有这个问题，同时如果输出不是 pdf 而是 png 或者 tiff 也没有这个问题，
所以问题出在`device`上。

```R
ggsave("~/Desktop/test.pdf", p, height = 2, device = cairo_pdf)
```

![正常的第一版](https://i.loli.net/2020/09/05/znVPXB1vFU7Ms5D.png)

好了，终于正常了，那么问题果然就是在 `device` 这个参数上。
比较一下用产生的PDF文件大小，`cairo_pdf` 占用空间 33k，
而 `pdf` 占用空间 8k，那么25k的差距大概就是原因了，前者内嵌了字体，而后者没有。

那么`pdf`用的究竟是什么字体？用软件查看了一下字体，是 Nimbus Sans. 
所以是 PDF 阅读软件没有找到 `Times New Roman` 然后用 Nimbus Sans 做了字体替换。
再仔细一看，不是`Times New Roman` 而是 `Times New Roman PS`，好了，破案了。
