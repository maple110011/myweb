
# 第三层地狱:失败的向量操作

我们到达了地狱的第三层，这里充满了无休止的冷雨，守卫在这里的刻耳柏洛斯(Cerberus)从它的三个喉咙中发出嘶吼，这层地狱中的亵渎者身着黄金制成的闪闪发光的服饰，并永恒地承受着能压垮他们的重量。在这儿维吉尔对我说：“记住一条法则——一个东西越完美，它的痛苦或快乐也就越极致。”

让我们来看一些代码

``` r
lsum <- 0
for(i in 1:length(x)) {
  lsum <- lsum + log(x[i])
}
```

别这样，这是在用C语言的写法(非常C语言的写法)在写R，我们可以用更简单的方式实现同样的功能

``` r
lsum <- sum(log(x))
```

这样不仅对你的手腕更友好，而且在运算速度上也更快(此外它还顺带解决了一个小问题——循环中若x长度为0时将报错)

向量化操作是上述代码得以起作用的关键，log函数按R的传统是向量化的运算，也即它会对向量中的每一个值做同样的操作，更具体地来说，这行代码:

``` r
log(c(23,67.1))
```

    ## [1] 3.135494 4.206184

和这行代码的结果是一样的

``` r
c(log(23),log(67.1))
```

    ## [1] 3.135494 4.206184

sum函数则是在另一种意义上进行向量操作，它接受一个向量作为输入，并且基于向量的所有元素返回求和结果，也就是说sum(x)实际上是

``` r
x[1] + x[2] + ... + x[length(x)]
```

prod函数和sum是类似的，只不过是返回乘积而非求和。值得一提的是，乘积经常容易发生上溢或者下溢(这算是第一层地狱的问题)，取对数化为求和通常是较为稳妥的方法。

你通常不需要费力自己进行向量化，R的很多东西天生就是向量化的。我们回看第<a href="#firstcircle"><strong>??</strong></a>章中用到的函数quadratic.formula，由于运算函数是向量化的，无论输入如何，输出结果都会是一个向量。唯一的小问题是每对输入都有两个答案，我们调用cbind就是为了把答案匹配起来。

在二元运算中，循环将会自动地在向量情况下发生，譬如

``` r
c(1,4) + 1:10
```

    ##  [1]  2  6  4  8  6 10  8 12 10 14

下面这个例子同时有着第<a href="#firstcircle"><strong>??</strong></a>章和第<a href="#secondcircle"><strong>??</strong></a>章的问题

``` r
ans <- NULL
for(i in 1:507980) {
  if(x[i] < 0) ans <- c(ans, y[i])
}
```

我们完全可以用非常简洁的方式实现它

``` r
ans <- y[x<0]
```

倍增的for循环操作通常是由于直接从其他语言把代码抄了过来，完全按照其他语言的内核照抄代码并不是一件好事，你最好思考一下R是怎么运行的，直接照抄其他语言的写法可能会让你觉得它们运行起来又快又好，但当你用R的思路重新整理代码之后，你将会发现R的强大之处(当然前提是你知道R的优势在哪，这样你才能写出R口音的代码)

如果你正在尝试对抄过来的代码(拥有很多for循环)实现R本土化，请想一想，如果你的函数不是向量化的，那么你可能需要使用Vectorize来得到一个向量化的版本，但这只是一种外在的向量化，而非编写固有的向量运算函数，Vectorize函数仍旧使用原始函数来执行循环。

一些函数可以接受函数作为参数，并要求它们是向量化的(除了outer和integrate)

下面给出了另一种向量运算的形式

``` r
max(2, 100, -4, 3, 230, 5)
```

    ## [1] 230

``` r
range(2, 100, -4, 3, 230, 5, c(4, -456, 9))
```

    ## [1] -456  230

这种向量运算是把参数的集合作为一个向量，这种形式不是你应该期望的，因为它本质上上来说是和R的内核是格格不入的——min,max,range,sum还有prod是少数例外。特别地，mean不遵守这种向量操作的形式，并且很不幸的是，当你错误地使用了这种形式时R并不会报错

``` r
mean(2, -100, -4, 3, -230, 5)
```

    ## [1] 2

你可以通过套上c函数来修正错误

``` r
mean(c(2, -100, -4, 3, -230, 5))
```

    ## [1] -54

一个使用向量操作的理由是这样算得更快，向量操作的本质是循环，而循环在C语言中必然比在R中跑得更快。某些情况下，这一点非常重要，而在另一些情况下则无伤大雅，譬如一个小循环在现在的计算机中无论是用R还是用C跑都差不了多少。

另一个使用向量操作的理由是更好懂，譬如:

``` r
volume <- width*depth*height
```

这条代码明确地展示了变量间的关系，无论我们需要计算一次还是一百万次，都不会伤其分毫。代码的可读性对高效工作来说是非常重要的，计算机的时间不值钱，但是我们的时间很值钱(并且晦涩的代码会带来挫败感)，这是Uwe Ligges的座右铭

`$$\boldsymbol{更好的电脑是廉价的，但有损思考能力}$$`
R语言的新手普遍会问同一个问题:“我该怎么为一些类似的对象分配名字呢？”你可以这样做，但最好别想要这样做——用向量化的思维会更好，把同类型的对象都放到列表里去，随后的分析和操作会更加顺滑。

## 索引操作(subscripting)

<div class="tabwid"><style>.cl-3e16a598{table-layout:auto;}.cl-3e061192{font-family:'Arial';font-size:11pt;font-weight:normal;font-style:normal;text-decoration:none;color:rgba(0, 0, 0, 1.00);background-color:transparent;}.cl-3e0b11ec{margin:0;text-align:center;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);padding-bottom:5pt;padding-top:5pt;padding-left:5pt;padding-right:5pt;line-height: 1;background-color:transparent;}.cl-3e0b2e5c{background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(102, 102, 102, 1.00);border-top: 1.5pt solid rgba(102, 102, 102, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-3e0b2e66{background-color:transparent;vertical-align: middle;border-bottom: 0 solid rgba(0, 0, 0, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}.cl-3e0b2e67{background-color:transparent;vertical-align: middle;border-bottom: 1.5pt solid rgba(102, 102, 102, 1.00);border-top: 0 solid rgba(0, 0, 0, 1.00);border-left: 0 solid rgba(0, 0, 0, 1.00);border-right: 0 solid rgba(0, 0, 0, 1.00);margin-bottom:0;margin-top:0;margin-left:0;margin-right:0;}</style><table data-quarto-disable-processing='true' class='cl-3e16a598'>
&#10;```
&#10;<caption style="display:table-caption;">(\#tab:t-1)<span>下标索引方法</span></caption>
&#10;```{=html}
&#10;<thead><tr style="overflow-wrap:break-word;"><th class="cl-3e0b2e5c"><p class="cl-3e0b11ec"><span class="cl-3e061192">索引类型</span></p></th><th class="cl-3e0b2e5c"><p class="cl-3e0b11ec"><span class="cl-3e061192">作用</span></p></th></tr></thead><tbody><tr style="overflow-wrap:break-word;"><td class="cl-3e0b2e66"><p class="cl-3e0b11ec"><span class="cl-3e061192">正值向量</span></p></td><td class="cl-3e0b2e66"><p class="cl-3e0b11ec"><span class="cl-3e061192">选择这些索引位置的值</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-3e0b2e66"><p class="cl-3e0b11ec"><span class="cl-3e061192">负值向量</span></p></td><td class="cl-3e0b2e66"><p class="cl-3e0b11ec"><span class="cl-3e061192">选择除了这些索引位置的其他值</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-3e0b2e66"><p class="cl-3e0b11ec"><span class="cl-3e061192">字符向量</span></p></td><td class="cl-3e0b2e66"><p class="cl-3e0b11ec"><span class="cl-3e061192">根据列名或行名进行选择</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-3e0b2e66"><p class="cl-3e0b11ec"><span class="cl-3e061192">布尔向量</span></p></td><td class="cl-3e0b2e66"><p class="cl-3e0b11ec"><span class="cl-3e061192">选择为TRUE和NA的位置对应的值</span></p></td></tr><tr style="overflow-wrap:break-word;"><td class="cl-3e0b2e67"><p class="cl-3e0b11ec"><span class="cl-3e061192">缺失</span></p></td><td class="cl-3e0b2e67"><p class="cl-3e0b11ec"><span class="cl-3e061192">选择所有值</span></p></td></tr></tbody></table></div>

下标操作在R中是非常有用的，并且通常能使得向量操作如虎添翼。表<a href="#tab:t-1"><strong>??</strong></a>对下标操作做了总结。

数组和数据框的维数索引是不一样的。数组(包括矩阵)可以用一个均为正数的矩阵进行索引，索引矩阵的列数要和数组的维数一样多——也就是说一个矩阵对应的索引矩阵有两列。索引得到的结果是一个包含选中项的向量而非数组。

列表也可以像向量一样被下标索引，然而，列表有着两种特殊的索引形式:“\$”和”\[\[“，它俩几乎是一样的，不同之处在于”\$“需要的是变量名而非字符串.

``` r
mylist <- list(aaa=1:5, bbb=letters)
mylist$aaa
```

    ## [1] 1 2 3 4 5

``` r
mylist[['aaa']]
```

    ## [1] 1 2 3 4 5

``` r
subv <- 'aaa'; mylist[[subv]]
```

    ## [1] 1 2 3 4 5

其实我刚刚说的有一些小问题，事实上用”\[\[“也可以对向量进行索引操作，当你只需要一项时，这可能是一个更稳妥的选择。不过如果你使用”\[\[“进行索引却想要不止一项，那结果就不尽如人意了。

## 条件语句

## 不可向量化的操作
