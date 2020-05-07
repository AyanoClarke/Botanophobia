+++

title = "ggplot2 代码阅读 (Part 0)"
description = "关于这个系列的走向以及对 `ggplot` 的介绍 "
date=2020-05-08

[taxonomies]
categories = ["这是什么"]
tags = ["ggplot2", "R"]

[extra]
author = "Clarke"
relative_posts=[]

+++



R 语言中绘图的程辑包中最有名气大概就是 ggplot2 了 ，为了更好理解 ggplot2 我们来阅读一下 ggplot2 的一些代码吧。如果是想学习基础的 ggplot2 的使用方法的话，可以读一下这本 [ggplot2: Elegant Graphics for Data Analysis](https://ggplot2-book.org/) ，而这一个系列的博客将会基于 `v3` 的 ggplot2 阅读代码，主要给 ggplot2 的源代码加上注释和一些说明。 

再次说明一下，这不是一个 gpplot2 的教程，而是一个稍微看一点代码，但又不过分深入的悄悄划水的系列。对 R 语言的基本要求大概就是知道面向对象的相关知识，还有环境 (environment) 的概念，这样就可以很轻松划过去了。如果不了解的话可以去阅读一下 [Advanced R](https://adv-r.hadley.nz/) 中相关章节，也非常快能够理解了。

首先来介绍一下 ggplot2 吧，这是一个超过 10 年的绘图包，作者主要是被誉为改变了 R 语言的男人 [Hadley Wickham](http://hadley.nz/) 以及 [Winston Chang](https://github.com/wch)，引用一下这个包的定位。
> ggplot2 is a system for declaratively creating graphics, based on [The Grammar of Graphics](http://amzn.to/2ef1eWp). You provide the data, tell ggplot2 how to map variables to aesthetics, what graphical primitives to use, and it takes care of the details.

正如许多介绍里说的，这个程辑包是基于低级别的绘图 `grid` 实现了 Leland Wilkinson 的 The Grammar of Graphics。所以需要涉及到 `grid` 的一些知识，这一部分也可以通过  [R Graphics](https://www.amazon.co.jp/Graphics-Third-Chapman-Hall-English-ebook/dp/B07KRP5M4P/ref=sr_1_7?__mk_ja_JP=%E3%82%AB%E3%82%BF%E3%82%AB%E3%83%8A&dchild=1&keywords=R+Graphics+%28Chapman+%26+Hall%2FCRC+The+R+Series%29&qid=1588876058&s=english-books&sr=1-7)  获得相关的知识。

用 ggplot2 来绘图的时候我们主要经历这些步骤：
1. 确定数据

2. 创建一个绘图对象 `ggplot`

3. 确定很多绘图图层 `geom` ，同时确定数据如何映射，变化成为图形。

然后我们就可以得到一个包含了如何绘制图信息的对象，然后我们可以绘制，也可以保存。而这次的主要来看 `ggplot` 这个部分的代码。

## 绘图时候的类`ggplot` 
`ggplot` 是 ggplot2 中的一个 `S3` 类，绘画时候将一些绘画需要的信息都存储在这个类里。它的创建方法为：

```r
# 代码位于 R/plot.r
ggplot.default <- function(data = NULL, mapping = aes(), ..., environment = parent.frame()) {
  
  # ...
  
  data <- fortify(data, ...)

  p <- structure(list(
    data = data,
    layers = list(),
    scales = scales_list(),
    mapping = mapping,
    theme = list(),
    coordinates = coord_cartesian(default = TRUE),
    facet = facet_null(),
    plot_env = environment
  ), class = c("gg", "ggplot"))

  p$labels <- make_labels(mapping)

  set_last_plot(p)
  p
}
```

从代码里面可以看出，一个 `ggplot` 类的对象拥有这些属性：`data` ， `layers` ，`scales`， `mapping` ， `theme` ，`coordinates`，  `facet` ，`plot_env` 。如果熟悉用 ggplot2 做图的画，对上面的几个属性应该不会陌生。当我们拥有 `ggplot` 类对象的时候，ggplot2 实现了用加法方式，将不同的 `gg` 类相加生成新的 `ggplot` 类对象，再调用 `print` 和 `plot` 的方法就可以绘图了。 而 `plot` 的内容和 `print` 一样，所以直接来看 `print`。

```r
# 代码位于 R/plot.r
print.ggplot <- function(x, newpage = is.null(vp), vp = NULL, ...) {

  set_last_plot(x)
  if (newpage) grid.newpage()
  # Record dependency on 'ggplot2' on the display list
  # (AFTER grid.newpage())
  grDevices::recordGraphics(
    requireNamespace("ggplot2", quietly = TRUE),
    list(),
    getNamespace("ggplot2")
  )
  # data 存储了图层的数据
  data <- ggplot_build(x)
  # gtable 存储了图像信息，用 ggplot_gtable 转化 data 获得
  gtable <- ggplot_gtable(data)
  # 调用 grid
  if (is.null(vp)) {
    grid.draw(gtable)
  } else {
    if (is.character(vp)) seekViewport(vp) else pushViewport(vp)
    grid.draw(gtable)
    upViewport()
  }
  # 为了满足 tidyverse 里面的管线操作，返回一个不打印的 x 
  invisible(x)
}
```

在这里主要包括了四个函数
1. `fortify()` 这个是帮忙转化数据成为 `data.frame`
2. `make_labels()` 将 `mapping` 转化成标签
3. `set_last_plot()` 记录上一次的绘画结果
4. `ggplot_build()` 用 `ggplot` 对象输出两个部分，一个是每个图层的数据，以及一个包含坐标界限等的面板信息。
5. `ggplot_gtable()` 将 `ggplot_build` 对象转化成能被 grid 绘画的 `grob` 对象。

`fortify` 目前的实现方法可能会被替代，`make_labels()` 涉及到映射相关，现在先放着，这一次我们先来阅读后面三个函数。

### 用 `.store` 来查找最后一次绘图
这一部分主要用于存储最后一次绘图的信息，可以用于保存图像啊之类的功效。
将信息保存在全局的 `.store` 里的 `.last_plot` 里面，提供 `set_last_plot` 以及 `last_plot` 可以调用和设定最后一次绘图。

在看这段之前，要时刻牢记着 R 里面的这类包含着参数，本体，环境的函数是一个由 `function()` 创建出来的对象，即 `function(arg1, arg2) {body}` 就是 ```(`function`(alist(arg1, arg2), body, env))```，这样就很好理解为什么可以这么做。图形的数据保存在 `.last_plot` 中间，不能直接修改，需要通过 `get` 和 `set` 才行。

```r
# 代码位于 R/plot-last.r
.plot_store <- function() {
  .last_plot <- NULL

  list(
    get = function() .last_plot,
    set = function(value) .last_plot <<- value
  )
}
.store <- .plot_store()
```

有了这个定义之后，那么 `set_last_plot` 和 `last_plot` 就非常简单了。

```r
# 代码位于 R/plot-last.r
#' Set the last plot to be fetched by lastplot()
set_last_plot <- function(value) .store$set(value)

#' Retrieve the last plot to be modified or created.
last_plot <- function() .store$get()
```

顺便我们也可以看一下 `ggsave` 所接受的参数

```r
# 代码位于 R/save.r
ggsave <- function(filename, plot = last_plot(), device = NULL, path = NULL, scale = 1, width = NA, height = NA, units = c("in", "cm", "mm"), dpi = 300, limitsize = TRUE, ...) {#### 函数本体 ####}
```

可以看到在 `ggsave` 里面的默认传参就是 `last_plot()`。

### `ggplot_build` 与 `ggplot_gtable` 获得 grid 需要的数据 `gtable`
正如最前面所说的，ggplot2 是建立在 grid 之上的，所以，这两个函数就是主要用于产生可以用于 grid 作画的数据结构 `gtable` ，也就是 `grob table`。

`ggplot_build` 也是一个 `S3` 类。这个注释做得不要太清楚，基本上每一步都有清晰的注释，除去那些与 `ggplot` 类中的属性的相关函数，比如计算 `layout` ，基本就是按照前面提供的步骤走，去构建出一个可以用来画图的 `data` 。

```r
# 代码位于 R/plot_build.r
ggplot_build.ggplot <- function(plot) {
  plot <- plot_clone(plot)
  if (length(plot$layers) == 0) {
    plot <- plot + geom_blank()
  }

  layers <- plot$layers
  # 这部分的 layer 以后再说啦
  layer_data <- lapply(layers, function(y) y$layer_data(plot$data))

  scales <- plot$scales

  # 这会经常用，同时由于早期涉及上的失误，layer 这里有挺大改进空间
  # 只不过动力可能不足了。
  # Apply function to layer and matching data
  by_layer <- function(f) {
    out <- vector("list", length(data))
    for (i in seq_along(data)) {
      out[[i]] <- f(l = layers[[i]], d = data[[i]])
    }
    out
  }

  # Allow all layers to make any final adjustments based
  # on raw input data and plot info
  data <- layer_data
  data <- by_layer(function(l, d) l$setup_layer(d, plot))

  # Initialise panels, add extra data for margins & missing faceting
  # variables, and add on a PANEL variable to data
  layout <- create_layout(plot$facet, plot$coordinates)
  data <- layout$setup(data, plot$data, plot$plot_env)

  # aesthetics mapping
  # ... 

  # statistics mapping
  # ...

  # scales
  # ...

  # Let Layout modify data before rendering
  data <- layout$finish_data(data)

  structure(
    list(data = data, layout = layout, plot = plot),
    class = "ggplot_built"
  )
}
```

然后就是一个超级长的 `ggplot_gtable()` ，由于内容涉及太多其它东西比如`ggproto` 的各种衍生，`guides`， `theme`等等，就暂时省略了，只看一个大概，全部内容就结合各个类来阅读。

```r
ggplot_gtable.ggplot_built <- function(data) {
  plot <- data$plot
  layout <- data$layout
  data <- data$data
  theme <- plot_theme(plot)

  geom_grobs <- Map(function(l, d) l$draw_geom(d, layout), plot$layers, data)
  layout$setup_panel_guides(plot$guides, plot$layers, plot$mapping)
  plot_table <- layout$render(geom_grobs, data, theme, plot$labels)

  # Legends
  # ...

  # Title, Subtitle, Tag, Caption
  # ...
  
  # Margins
  # ...

  plot_table
}
```

## 总结
这次是 part 0 嘛，所以比较水，不过大致通过 ggplot2 所有代码操作的最终目的 —— 构建一个 `ggplot` ，以及如何从 `ggplot` 变成图像的过程展现了一下。这个系列一共有多少期还没有确定，不过应该不会坑。