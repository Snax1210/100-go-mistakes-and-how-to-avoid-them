#  👀Data types 数据类型篇

## 🤔17.创建容易混淆的八进制字面量
![d330df27fcee3381f4cc79beef423286.png](:/4e718fa6b661470c8a342191d87a2806)
go中0开头的整型数字被认为是8进制数字，8进制的010是十进制的8，所以100+010是108
所以需要使用8进制的时候尽量用0o，0和0o效果是一样的，但是会使代码更易懂

## 🤔18.忽视整型溢出

## 🤔19.没理解浮点

## 🤔20.没理解切片slice的length和capacity
很多人容易把切片的length和capacity混淆
`s:=make([]int,3,6)`
第一个参数是length，第二个参数是capacity
![46636e946a1e4a54388bab4f84fe65d9.png](:/75403d6171e4446ba67b94c30a4b7041)
切片通过append插入元素，如果超过capacity大小，会创建一个新的array，将之前的容量翻倍，当元素超过1024时，容量每次增长25%，初始array不再被引用，如果在堆上分配就会被gc回收
s1,s2引用同一个array
![c1bc5d7902904f0ab3dd343bca53e31a.png](:/9c0d64b305074fb4a6c917b9ea9cecca)
修改某一个值，s1和s2中的值都会改变
![82093ab51d3bf9862a60a6024d5c3b46.png](:/a9754138b39a43ebabcb412b58981d43)
对s2 append新的值，s1不会被修改
![24054a11d86709f167c62ac90d00dfbe.png](:/f431a5b1fa7b4cbea74e23f4b1cc4d5a)
如果对s2 append超过容量的值，会创建新的数组，s1和s2指向不同的array
![7b99d8c11ab0823c7018fdb4be462a1a.png](:/d0f04ca0d25d4138ac4d3d6c4e6fb5e6)

## 🤔21.低效的切片初始化
用make初始化切片的时候尽量提供长度和容量，不然append值的时候会一直开辟新的空间并复制原来的array到新的array中，gc还需要努力回收，如果切片元素太大会降低性能
2种方式，一种分配cap不分配length，另一种分配length
![fd297e077634a5025ff5b0139027f60b.png](:/462cc6976a724d36b34ce7eef913371a)
![b1973cc68c4883baa367f312d864ca04.png](:/7f0e5cd8d8fb48db92a68b534db7f059)
![32e0f1758e74e8c48a0b16dc5711355f.png](:/81378293eeca4313a295cc12226ab55e)
虽然第三种更快，但是和第二种性能相差不大，代码却更复杂，推荐第二种用append
![62445a05b708f2f3c6d5ff8d3008718e.png](:/137a17641370475994fee58ee5be3782)

## 🤔22.对nil和empty的切片混淆不清
nil切片是空的，但空切片不一定是nil
nil slice没有任何内存分配
empty slice长度为0
![095cc3b0a4d863e3d0ff29ca663694c2.png](:/9162c6d9683246c38f19c41f0cd560e2)
![042b67a8188de0b598fb697650e518a9.png](:/0b8b074196e749dfbff56e3ff37981c0)
使用场景：
1.如果不确定切片长度以及切片是否能为空，使用 `var s []string`初始化
2.如果切片长度是已知的，就用`s=make([]string,length)`初始化
3.`s=[]string{}`在有初始化元素的时候使用


## 🤔23.检查slice为空的不恰当使用
不要用是否为nil判断切片是否为空，应该**取切片长度**来判断
错误示例：
![8862e6a2481de5c5fed6c6453df0ff00.png](:/32aa97b760ce4431a6c776ccb4de0f1d)
![d52b811e2590bd7af40ba6e759d3b5cf.png](:/8bd8051ef32b49308fe6a03b60545932)
检查长度
![3bf8453af4f7c7258b7eb74710d6689c.png](:/6c7ca2a1b30340b699144b1d70fcccaa)


## 🤔24.错误使用切片copy函数
错误示范：
![461f77afc2bb125bb4b021148b597ab5.png](:/4dd2c5b6b2954ce0bd5226f0e0a1977c)
解决：
1.![172f40509ac492a222ec02b44d368fa8.png](:/227b927c1d95462485b910ca37d877f1)
2.![87a6ae6c8aec85209acfbda6d03678e2.png](:/ab6e776fadb4489bbcd54bff6e0de60e)

## 🤔25.使用切片append产生的副作用
![ccb17186776087af475d6ff2b9a91281.png](:/52aabab15ed945d0b31469f8d79f09c8)
s1,s2,s3共享内存
![82124d453f4cc5a4bef0f8490b1600b3.png](:/3bfd4433d9444ff7898d87d50c0f1fa4)
![acb919956c58a1eecbc5eef891b03a82.png](:/44b85effb16f43e1ba46955d53826236)
解决方式：
1.使用copy，缺点是代码可读性差，以及slice很大时使用copy开销大
![67ae72362d8f853cc28ff0c2846fd4ad.png](:/ec33f222886142189682c95713961f83)
2.使用全切片表达式，从一个已有切片中创建新的切片
`newSlice=oldSlice[low:high:max]`
![7d8b08ade47487c04d9fb05183d56c61.png](:/a735ae29f7c0410e938cd0cc98883b78)
![761e8a0c886d13f3ff6da01404fa4122.png](:/b09e6457dce741cb99547cea521b8b2a)

## 🤔26.切片内存泄漏
场景示例1：一条消息包含100万字节，前5字节表示消息类型，getMessageType计算消息类型，运行程序消耗1G内存，剩余空间因为切片的引用无法被gc掉
![12c0442d741224b371139cd3f13f74ff.png](:/44ba919739f0400d87271ece0b7b655e)
![bbc116b4d572214e4d8e778404cbd9f9.png](:/4f2a457a40154f9f9f6d9a08e50730f0)
解决：
![06da0c7f856250eac94658f191dc2f23.png](:/d04bb6c119e542a4b3cc14f1e638bd71)
注意：这种情况最好也不要用全切片表达式，gc不会回收
场景示例2：
![dc0db78592e368eede706b6a25c47707.png](:/6ae3637f9c634067b08814708b82b427)
![a6894b3366ab2140db3f60dbd8029e1d.png](:/91172e7f2cd54ff1920518a89aca3d56)
解决：
1.使用copy，因为返回的不是原切片，没有被引用，所以gc能回收
![34afd3c0ab85d9b8979d1905ad04274f.png](:/6330014241c44a2bb9cb3477cbe1ed76)
2.如果想保留1000个元素的底层容量，可以显式地将其余切片标记为nil，这里返回len为2，cap为1000的切片，GC可以回收剩下的998个
![c22e7be857b2a0357cda7ee70bd30aa7.png](:/e598fd7d0a6e44bd88f4946da0679b7d)
![629b850746dfa8cce3e60622fdadc765.png](:/c852a42bb3ae4bb385a74ce1b35f59ed)
使用哪一种取决于保留的i的比例，用第一种方式，就必须从0迭代到i，用第二种方式就必须从i迭代到n

## 🤔27.无效的Map初始化
map增长时，桶的数量翻倍，出现map增长的情况：
1、负载因子大于常量值（目前是6.5）
2、桶溢出了（超过一个桶的数量）
插入一个元素时，最坏的情况下复杂度是O(n)，因为内存不够map增长时，所有的keys会被重新分配到所有的桶里
正确的map初始化：
指定capacity，避免大量的复制操作
![c08a14d51523609eb063c78f6f9805c1.png](:/0d45ccb62c7d4b7fbe6cbe3aba4cfd3e)


## 🤔28.Map的内存泄漏
下面代码：
1、分配一个空map
2、新增100万个元素
3、删除所有元素
![8601013b450c14322b21883087fcd3c5.png](:/6232138d6cc14f00bdbb2dec3848629e)
![67e5affb01b1c655c8e645001f9f7c02.png](:/bad5d99c04b640f29f0470045d722e8c)
![c714fa74b80c70dd20bc607f571e303c.png](:/300e7c8a7c644aaab50f76457407c647)
![9f13810a8cdcd70c3162e2e286b24abf.png](:/5699116d4c744736803f130ea0a2de91)
添加100万元素，hmap中B的值是18，有2^18=262144个桶，移除元素之后桶的数量还是这么多

删除元素后，堆大小还是没有缩减，因为map中的桶的数量不能减少，所以从map中删除元素不会影响现有的桶的数量，它只是将桶里的东西归零  （map只能不断增长，桶的数量只能增加，不能减少）

解决方案：
案例 如果map存储客户数据，平时的数据不多，在出现活动的时候用户数量激增，之后用户减少，100万个用户的数据很大
用指针存储value，即便删除元素桶的数量还是那么多，但是每个桶条目为值保留一个指针大小，而不是128byte
map[int][128]byte    ---->  map[int]*[128]byte


## 🤔29.错误的值比较
可操作符直接比较的数据类型：comparable类型
bool
整型
浮点
strings
channels
interfaces
pointers
structs
arrays

不可比较类型：
比较方式1：使用reflect包 中的reflect.DeepEqual
接收的数据类型：arrays, structs, slices, maps, pointers, interfaces, and functions
![ff4bc1c9e1f319a3ac6f99b2c698a508.png](:/ed571f2f422f4508a8d18f6a363e0e0b)
缺点：性能比操作符低，因为内部要花时间比较
比较方式2：自定义，如果对性能要求更高，这是个最好的解决方案
![250de1ca0598707f4e602f6cd8e86cfa.png](:/48ff76a20df24196aca3f277921dbec8)
