+++

title = "啊，我亲爱的大头菜"
description = "在谷歌表格中记录与预测大头菜的价格"
date=2020-04-06

[taxonomies]
categories = ["瞎整"]
tags = ["Google Sheets", "Animal Crossing", "大头菜"]

[extra]
author = "Clarke"
relative_posts=[]

+++

沉迷在《集合啦！动物森友会》的我沉迷在炒~~股~~大头菜之中。大头菜的预测模型其实并不复杂，对应的也有一个[网站](https://juo6442.github.io/moothumb/)，但是由于每次都要输入大头菜的价格，健忘的我想把他保存在表格之中。谷歌表格是我心中最好的选择了，在云端的他小巧可爱，而且可以用近似于 javascript 的 Google Apps Scripts 来写脚本，就可以直接把项目抄到谷歌表格中了。这篇是记录我是怎么抄的。共享表格的链接是[这个](https://docs.google.com/spreadsheets/d/1bfOk0q9uTq6uUqulqf_hfb9x0KQSO575hpSLNS33m8w/edit?usp=sharing)，直接点进去复制一份就好了。

## 环境

因为谷歌的云编辑器既没有折叠，也没有提示，是一个名副其实的文本编辑器，所以为了方便复制代码，于是决定在本地用 VS Code 来写。

首先是在本地安装 [clasp](https://github.com/google/clasp)，并且关联谷歌帐号，这样就可以把写完的脚本直接上传。

```shell
npm install -g @google/clasp
clasp login
```

接下来创建一个项目，并且为了在 VS Code 里面使用补全。

```shell
npm init
npm install --save @types/google-apps-script
```

通过`clasp create` 创建新的表格，这些信息会被记录在

```
...
├── .clasp.json
└── appsscript.json
```

这两个文件之中，创建新文件然后在写完全部代码后，使用`clasp push` 就可以把代码上载，也可以用 `clasp open` 快速打开。

## 抄

原来的代码在[这里](https://github.com/juo6442/moothumb/tree/acnh_params)，通过阅读代码可以发现 `src/js/predictor.js` 里面的 `predict` 函数是主要的出口。`src/js/presets.js` 里面保存大头菜价格浮动的参数。结合一下 [wiki](https://appmedia.jp/atsumare_doubutsunomori/4626808) 中大头菜的小知识，去快速阅读代码，就很容易懂大头菜的四种涨幅类型。然后原码通过 `src/js/ui.js` 把预测得到的结果表示出来。

好了，预测部分直接复制粘贴，显示结果稍作修改，改成由谷歌表格提供的接口即可。

## 改

首先为了让谷歌表格在每次记录数据更改的时候运行脚本，所以要提供一个接口函数，这里把它叫做 `runPredict`，接下来需要读取已有的数据，这里用接口读取反而比原来填表格要方便，读取完数据后，运行上述所说的`predict`，然后将预测结果写出成新的表格。直接调用`GoogleAppsScript.Spreadsheet.Sheet.clear()`强制刷新比较方便，然后重新绘制，谷歌表格提供足够丰富的接口真是太快乐了。

最后加上一个只要更改就运行`runPredict`函数，一切就完成了，从开始打算改到阅读，抄，改完，一共只花了一个多小时。考虑到网站的全平台性，以及 Google App Scripts 的易上手，易抄，以后有什么需要不如考虑考虑谷歌表格吧~

