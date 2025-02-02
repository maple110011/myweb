# 第一层地狱:落入浮点数陷阱 {#firstcircle}
一旦我们穿过了阿切伦河,我们就到达了地狱的第一层,这里是善良的异教徒的家.这些异教徒生活在对浮点神的无知之中,他们期待下面的代码将返回TRUE

``` r
0.1==0.3/3
```

```
## [1] FALSE
```
他们还期待下面这一代码返回的结果中会出现一个TRUE

``` r
seq(0,1,by=0.1)==0.3
```

```
##  [1] FALSE FALSE FALSE FALSE FALSE FALSE FALSE FALSE FALSE FALSE FALSE
```
但是决不应该认为下面这个向量仅是一个值的重复

``` r
unique(c(0.3,0.4-0.1,0.5-0.2,0.6-0.3,0.7-0.4))
```

```
## [1] 0.3 0.3 0.3
```
我在古早时期写下了我的第一行代码,它的作用是求解二次方程.古早时期的意思是没有显示屏,只有打孔器,而打孔器是没有退格键的——也即一旦孔已经打出来了,就没法再把它们填上了.因此,任何一行代码中的小错误都意味着整条孔带都没用了,得从头开始运行代码,这个过程我真是不能更熟悉了.

经历了漫长的折磨之后,当我得到一沓正确的孔带时我终于可以一扫阴霾了.然而欢乐时光总是短暂的,下一个步骤是把这沓孔带放进计算机操作员会查看的框里,几个小时后,结果的（大）纸质输出将被放在文件架上,令人感慨且理所当然的是,我的代码中依然存在着一个错误,在经过另一番与打孔机的较量之后（这次相对短暂）,孔带终于又放到框中去了.

仅经历了不多的迭代,我就意识到了它只告诉我过它出现的第一个错误.到了第三天,输出结果终于没有关于出错的信息了,呈现在上面的只有一个答案,一个错误的答案.我求解的问题是一个很简单的二次方程,答案很明显地是2和3,而我的程序告诉我答案是1.999997和3.000001,在经受了这么多的痛苦之后,它甚至没法算出正确的答案.

我们可以写一个R函数来更快地求解二次方程

``` r
quadratic.formula <- function (a, b, c){
  rad <- b^2 - 4*a*c
  if (is.complex(rad) || all(rad >= 0)){
    rad <- sqrt(rad)
  }else{
    rad <- sqrt(as.complex(rad))
  }
  cbind(-b-rad,-b+rad)/(2*a)
}
quadratic.formula(1,-5,6)
```

```
##      [,1] [,2]
## [1,]    2    3
```

``` r
quadratic.formula(1,c(-5, 1),6)
```

```
##                [,1]           [,2]
## [1,]  2.0+0.000000i  3.0+0.000000i
## [2,] -0.5-2.397916i -0.5+2.397916i
```
这比旧的编程方式更为一般,并且更为重要的是它得到了正确的答案2和3.不过事实上它并没有做到.R只是在打印结果上使得大部分数值误差不可见了,我们可以通过用它减去正确答案来看看它实际上有多么错误：

``` r
quadratic.formula(1,-5,6)-c(2,3)
```

```
##      [,1] [,2]
## [1,]    0    0
```
好吧,这个问题下它得到了正确答案,但是只要我们稍微改动一点,误差就会出现了:

``` r
quadratic.formula(1/3, -5/3, 6/3)
```

```
##      [,1] [,2]
## [1,]    2    3
```

``` r
print(quadratic.formula(1/3, -5/3, 6/3), digits=16)
```

```
##                   [,1]              [,2]
## [1,] 1.999999999999999 3.000000000000001
```

``` r
quadratic.formula(1/3, -5/3, 6/3) - c(2, 3)
```

```
##               [,1]         [,2]
## [1,] -1.110223e-15 1.332268e-15
```
R的打印结果看起来非常好,这是一种祝福,也是一种诅咒,R对于把结果中的数值误差在打印时隐藏起来做的非常好,以致于我们很容易忘记这个问题,记住永远不要忘记这个问题

无论何时进行浮点运算,即使是非常简单的情况,也应该假定存在数值误差,如果偶然没有出错,那只是一个幸福的意外,而非你应得的.记住如果你想比较两个浮点数是否相等,使用all.equal函数而不是`==`

不要把数值误差(numerical error)和计算错误(error)混淆,计算错误指的是计算执行错误时出现的错误,数值误差指的是数字的有限表示产生的可见噪声.譬如当1/3被表示为33%时出现的是数值误差而非错误.

我们已经看到了异教徒对浮点神之信仰的暗面——打印的结果就是一切

``` r
7/13-3/31
```

```
## [1] 0.4416873
```
R默认的打印结果是一个方便的缩写,而非R中关于这数值的一切

``` r
print(7/13-3/31,digits = 16)
```

```
## [1] 0.4416873449131513
```
许多摘要函数打印出来的结果甚至更为受限

``` r
summary(7/13-3/31)
```

```
##    Min. 1st Qu.  Median    Mean 3rd Qu.    Max. 
##  0.4417  0.4417  0.4417  0.4417  0.4417  0.4417
```
根源于数字的有限表达的数值错误不仅会模糊答案,还会模糊我们的问题,在数学上矩阵的秩是一些确定的整数,而在计算机中,矩阵的秩是一个含糊的概念,由于特征值不需要明显为零或是明显非零,因此秩也不必是一个确定的数.

我们已走到第一层地狱的边缘,咬牙切齿的米诺斯是这里的守卫.他用尾巴缠绕自己的次数标志着他面前的罪人的水平.
