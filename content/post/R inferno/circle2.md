# 第二层地狱:增长对象 {#secondcircle}
我们来到地狱的第二层,这里生活着贪婪者.

让我们看看三种创建同一个数列的方法,方法一是增长对象(growing object)

``` r
vec <- numeric(0);n <- 100
for(i in 1:100) vec <- c(vec, i)
```
方法二是先创建一个对应长度的向量,然后按索引依次改变其中的值

``` r
vec <- numeric(n);n <- 100
for(i in 1:n) vec[i] <- i
```
方法三是用R的冒号操作符

``` r
n <- 100
vec <- 1:n
```


Table: <span id="tab:s-1"></span>Table 1: 三种创建序列方法的用时(秒)

| 序列长度| 增长样本 | 索引修改 | 冒号操作 |
|--------:|:--------:|:--------:|:--------:|
|     1000|   0.01   |   0.01   | 0.00006  |
|    10000|   0.59   |   0.09   | 0.00040  |
|   100000|  133.68  |   0.79   | 0.00500  |
|  1000000| 18718.00 |   8.10   | 0.09700  |


表<a href="#tab:s-1">1</a>显示了这三种方法在特定机器上生成长度为n的序列所用的时间(以秒为单位).三种方法所耗费的时间在对数-对数尺度上都大致与长度n呈线性关系,但增长系数有很大的不同

你可能想知道为什么增长对象方法如此之慢,它在计算上来说可以类比为郊区开发(suburbanization),当我们需要变化序列的长度时,序列在计算机中的位置可能没有足够的空间让它增加长度,那么计算机就需要把序列移动到一个更宽阔的空间去,下一次增加序列长度时空间又不够了,计算机就需要再次移动它的位置,这将花费相当多的时间.正如现实中的郊区开发一样,增长对象方法会耗尽所有可用的空间,最后只会剩下一些小内存空间,这被叫做内存碎片化(fragmenting memory)

对贪婪者而言,在rbind上使用增长对象方法是一个更为普遍,同时也更为危险的情景,例如:

``` r
my.df <- data.frame(a=character(0),b=numeric(0))
for(i in 1:n) {
  my.df <- rbind(my.df, data.frame(a=sample(letters, 1),
                                   b=runif(1)))
}
```
写出这种代码的主要原因可能是每一次观测都会得到不同值,也就是说,这段代码实际上更可能是

``` r
my.df <- data.frame(a=character(0),b=numeric(0))
for(i in 1:n) {
  this.N <- rpois(1, 10)
  my.df <- rbind(my.df, data.frame(a=sample(letters,this.N, replace=TRUE),b=runif(this.N)))
}
```
通常情况下,我们最终想要的那个对象是可以确定一个合理的大小上界的,这样的话我们应当首先生成该大小的对象,最后再把多余的值全都删掉.如果最终大小是未知的,那么也应该沿用这一准则,仅分段使用增长对象方法.


``` r
current.N <- 10 * n
my.df <- data.frame(a=character(current.N),b=numeric(current.N))
count <- 0
for(i in 1:n) {
  this.N <- rpois(1, 10)
  if(count + this.N > current.N) {
    old.df <- my.df
    current.N <- round(1.5 * (current.N + this.N))
    my.df <- data.frame(a=character(current.N),
                        b=numeric(current.N))
    my.df[1:count,] <- old.df[1:count, ]
    }
  my.df[count + 1:this.N,] <- data.frame(a=sample(letters,this.N, replace=TRUE),b=runif(this.N))
  count <- count + this.N
  }
my.df <- my.df[1:count,]
```
不过这样的问题一般都可以用一个更简便的方式解决——将不同的结果放入列表之中,然后再把它们合并到一块去

``` r
my.list <- vector('list',n)
for(i in 1:n) {
  this.N <- rpois(1, 10)
  my.list[[i]] <- data.frame(a=sample(letters,this.N,replace=TRUE), b=runif(this.N))
  }
my.df <- do.call('rbind', my.list)
```
有许多看似聪明的写法可以掩盖你正在使用增长对象方法的事实,譬如下面的例子:

``` r
hit <- NA
for(i in 1:1000000) {
  if(runif(1) < 0.3) hit[i] <- TRUE
  }
```
仅当条件为真时,hit才会增长

避免使用增长对象方法可能是R中最简单也最激励人心的加速代码运行路径

如果你占用了太多内存空间,R会抱怨的.关键问题在于R将所有数据都存在了内存之中,如果你的数据很大,这会是一个弊端,不过从灵活性的角度来说,这使得R对数据几乎没有任何限制

一些人可能会对下面这条报错信息可能非常熟悉

Error: cannot allocate vector of size 79.8 Mb.

这通常会带来一种误解:“我有xxxGB的内存,为什么R连80MB的空间都无法分配”,这事实上是因为R已经成功分配了相当多的内存,这条报错信息指的是R在某个小节点上想要分配内存时空间不足,而非对整个任务分配内存时空间不足.

自然地,每一个看到这条报错信息的人都会想:“我该做点什么来解决问题”,这儿有一些很简单的办法:
1.不要成为贪婪者,也即不要使用糟糕的语法结构(如增长对象方法)
2.换一个更好的电脑
3.减小问题的数量级

如果你违反了第一条,并且没法做到第二条或第三条,那你操作的余地就所剩无几了,要么重启你的R(这通常是无济于事的),要么就老老实实地优化你的代码,找出哪些地方使用了增长对象方法.一种可行的办法是在你的程序中插入下面这串代码


``` r
cat('point 1 mem', memory.size(), memory.size(max=TRUE), '\n')
```

这显示了R当前拥有的内存以及可用的最大内存

不过,一个可能更有效且更能提供信息的方法是使用Rprof和内存分析,值得一提的是Rprof还会给出程序的运行时间

另一种为内存减负的方法是将数据存储在数据库中,并根据具体需求提取相应的数据.虽然部署数据库会花费一些时间,但它会是一种相当自然的工作流程

在仅使用R的情况下构建“数据库”可以使用save函数,先将我们的数据分别储存为独立的文件,在日后使用时只读取其中的一部分,具体实现起来可能会像下面这样

``` r
for(i in 1:n) {
  objname <- paste('obj.', i, sep='')
  load(paste(objname, '.rda', sep=''))
  the_obj <- get(objname)
  rm(list=objname)
  # use the obj
}
```
随着计算机技术的发展,日后性能更好的计算机可以解决我们现在面临的问题吗？对一部分人来说是的,他们的数据集几乎总是一个大小,更好的计算机足以让他们轻松解决内存问题,但对另一部分人来说,更好的计算机意味着他们可以搬出更大的数据集,问题并不会得到解决.如果你属于后一种人,你可能得现在就开始熟悉如何在工作中利用数据库技术了.

如果你有一台内存超级大的计算机,你完全可以尝试一些R可以进行的“大操作”,通过下面的代码查看R的极限

``` r
?"Memory-limits"
```

