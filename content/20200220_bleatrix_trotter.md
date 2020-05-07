+++

title = "Bleatrix Trotter 另类数羊"
description = "一道题目的证明"
date=2020-02-28

[taxonomies]
categories = ["练习"]
tags = ["练习"]

[extra]
author = "Clarke"
relative_posts=[]

+++

今天在做 [codewars](https://www.codewars.com/) 的时候遇到一道[数羊题](https://www.codewars.com/kata/59245b3c794d54b06600002a)，题目描述了一种奇怪的数羊策略，首先给定数字 \\(N\\)，然后从 \\(N\\) 开始沿着 \\(2 \times N, 3\times N \dots\\) 这样数。每次数数的时候，会把这个数的每个数字记下来，比如 \\( 1692 \\) ，就会记下 \\([1,6,9,2]\\)，一直数到所有的数字都出现结束，比如 \\(1692, 2 \times 1692 = 3384, 3\times 1792 = 5076\\)，结束。题目是输入数字 \\(N\\)，返回最后数完的数字 \\(i \times N\\)，如果永远无法结束，则返回 \\( -1\\)。

题目的很简单，就是不断循环，直到全部得到退出。

```rust
use std::collections::HashSet;

fn trotter(n: i32)-> i32{
  const MAX = 1000;
  let mut root = n.to_string().chars().collect::<HashSet<char>>();
  for i in 1..MAX {
      let num = i * n;
      let _null = num.to_string().chars().map(|c| root.insert(c) ).collect::<HashSet<bool>>();
      if root.len() == 10 {
          return num;
      }
  }
  -1
}
```

问题在于是否真的对于 \\(N \ne 0 \\) 存在一个数，也就是代码里的  `MAX` ，能保证不返回 \\(-1\\) (即不存在)。

如果是在高中，我可能会用数论之类的来解决，但是经过大学四年生物的摧残，我可做不到了，好在有计算机可以辅助证明，先说结论，存在的。

分类讨论：

1. 显然，若\\(N\\)末尾数字为 \\(1 或者 9\\)，都是可以结束的，最多 \\(9\\) 次。

2. 因为 \\( 3 \times 7 \equiv 1\\) ，所以末尾数字为 \\(3, 7\\) 都是可以结束的，粗略最多 \\( 27\\) 次。

3. 进一步，若\\(N\\)末尾数字为 \\(0X, (X = 1\dots 9) \\)，都是可以达成的。粗略最多 \\( 45 \\) 次就可以达成。

4. 接下来是末尾数字为 \\(AB | B \equiv 0 \mod 2\\) 的情况：

   1. 末尾数字为 \\(4, 6, 8\\) 的数字，都可以转变成末尾数字 \\(2\\) 的情况，最多 \\(9\\) 次。

   2. 末尾数字为 \\(2\\) 的时候，就用穷举法。

      | Num  | Times | Final |
      | ---- | ----- | ----- |
      | 12   | 9     | 02    |
      | 22   | 14    | 08    |
      | 32   | 19    | 08    |
      | 42   | 12    | 04    |
      | 52   | 2     | 04    |
      | 62   | 13    | 06    |
      | 72   | 7     | 04    |
      | 82   | 5     | 10    |
      | 92   | 12    | 04    |

      就可以发现结合情况 4.1 都可以转化成末尾数字为 \\(02\\) 的情况。

5. 最后只剩下末尾数字为 \\(5, 0\\) 的情况了，当然注意到 \\(5 \times 2 \equiv 0 \mod 10\\),  这两个都是一样的，我们只要不停对数字除 \\( 10 \\)，就可以得到末尾最后非零 ，再用上述的情况处理。

最后可以粗略估计出一个上界，\\( 855 \\)，所以程序里面的 `MAX`，只要迭代 \\( 1000 \\) 次就可以了。