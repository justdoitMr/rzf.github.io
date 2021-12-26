# Bloom Filter 概念

布隆过滤器（英语：Bloom Filter）是1970年由一个叫布隆的小伙子提出的。它实际上是一个很长的二进制向量和一系列随机映射函数。布隆过滤器可以用于检索一个元素是否在一个集合中。它的优点是空间效率和查询时间都远远超过一般的算法，缺点是有一定的误识别率和删除困难。

# Bloom Filter 原理

布隆过滤器的原理是，当一个元素被加入集合时，通过**K个散列函数**将这个元素映射成一个位数组中的K个点，把它们置为1。检索时，我们只要看看这些点是不是都是1就（大约）知道集合中有没有它了：如果这些点有任何一个0，则被检元素一定不在；如果都是1，则被检元素很可能在。这就是布隆过滤器的基本思想。

Bloom Filter跟单哈希函数Bit-Map不同之处在于：Bloom Filter使用了k个哈希函数，每个字符串跟k个bit对应。从而降低了冲突的概率。

![1640434598754](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/25/201639-791645.png)

# 缓存穿透

![1640434627733](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/25/201714-327001.png)

`每次查询都会直接打到DB`

我们先把我们数据库的数据都加载到我们的过滤器中，比如数据库的id现在有：1、2、3

那就用id：1 为例子他在上图中经过三次hash之后，把三次原本值0的地方改为1

下次数据进来查询的时候如果id的值是1，那么我就把1拿去三次hash 发现三次hash的值，跟上面的三个位置完全一样，那就能证明过滤器中有1的

反之如果不一样就说明不存在了

那应用的场景在哪里呢？一般我们都会用来防止缓存击穿

简单来说就是你数据库的id都是1开始然后自增的，那我知道你接口是通过id查询的，我就拿负数去查询，这个时候，会发现缓存里面没这个数据，我又去数据库查也没有，一个请求这样，100个，1000个，10000个呢？你的DB基本上就扛不住了，如果在缓存里面加上这个，是不是就不存在了，你判断没这个数据就不去查了，直接return一个数据为空不就好了嘛。

# Bloom Filter的缺点

bloom filter之所以能做到在时间和空间上的效率比较高，是因为牺牲了判断的**准确率、删除的便利性**

- 存在误判，可能要查到的元素并没有在容器中，但是hash之后得到的k个位置上值都是1。如果bloom filter中存储的是黑名单，那么可以通过建立一个白名单来存储可能会误判的元素。
- 删除困难。一个放入容器的元素映射到bit数组的k个位置上是1，删除的时候不能简单的直接置为0，可能会影响其他元素的判断。可以采用[Counting Bloom Filter](https://link.juejin.cn?target=http%3A%2F%2Fwiki.corp.qunar.com%2Fconfluence%2Fdownload%2Fattachments%2F199003276%2FUS9740797.pdf%3Fversion%3D1%26modificationDate%3D1526538500000%26api%3Dv2)

# Bloom Filter 实现

布隆过滤器有许多实现与优化，Guava中就提供了一种Bloom Filter的实现。

在使用bloom filter时，绕不过的两点是预估数据量n以及期望的误判率fpp，

在实现bloom filter时，绕不过的两点就是hash函数的选取以及bit数组的大小。

对于一个确定的场景，我们预估要存的数据量为n，期望的误判率为fpp，然后需要计算我们需要的Bit数组的大小m，以及hash函数的个数k，并选择hash函数

## Bit数组大小选择

根据预估数据量n以及误判率fpp，bit数组大小的m的计算方式：

![1640434816460](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/25/202017-357281.png)

## 哈希函数选择

由预估数据量n以及bit数组长度m，可以得到一个hash函数的个数k：

![1640434842560](https://tprzfbucket.oss-cn-beijing.aliyuncs.com/hadoop/202112/25/202054-207477.png)

哈希函数的选择对性能的影响应该是很大的，一个好的哈希函数要能近似等概率的将字符串映射到各个Bit。选择k个不同的哈希函数比较麻烦，一种简单的方法是选择一个哈希函数，然后送入k个不同的参数。

哈希函数个数k、位数组大小m、加入的字符串数量n的关系可以参考[Bloom Filters - the math](https://link.juejin.cn?target=http%3A%2F%2Fpages.cs.wisc.edu%2F~cao%2Fpapers%2Fsummary-cache%2Fnode8.html)，[Bloom_filter-wikipedia](https://link.juejin.cn?target=https%3A%2F%2Fen.wikipedia.org%2Fwiki%2FBloom_filter)

> 布隆过滤器主要是在回答道缓存穿透的时候引出来的