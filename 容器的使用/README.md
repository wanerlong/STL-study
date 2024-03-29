STL各组件应用示例

STL六大组件

STL六大组件包括容器(container)、分配器(allocator)、算法(algorithm)、迭代器(iterator)、适配器(adapter)和仿函数(functor).
![图片1](https://user-images.githubusercontent.com/72439295/116250544-64a3e280-a7a0-11eb-962a-1be401ad4c67.png)
容器

STL中的容器大体分为序列容器、关联容器和无序容器.
![图片2](https://user-images.githubusercontent.com/72439295/116250928-c5331f80-a7a0-11eb-8d46-1e50aac50bb7.png)

vector底层是一段连续的内存空间,当容器满时进行扩容,将容器大小扩容为原来的两倍.

list类本身具有sort方法,容器本身实现的sort的性能一般比标准库中的算法sort更好.

multiset和multimap底层是使用红黑树实现的.

multimap支持重复的key,因此不能使用重载的[],其他map键值唯一.

unordered_multiset和unordered_multimap的元素个数小于篮子数目,若元素数目达到篮子个数,则容器扩容,将篮子数组扩充约一倍.

分配器

STL容器默认的分配器是std::allocator,除此之外gcc额外定义了几个分配器,其头文件均在目录ext下.
![图片3](https://user-images.githubusercontent.com/72439295/116251663-73d76000-a7a1-11eb-910e-025e9b59010d.png)

VC6.0的默认分配器只是对::operator new和::operator delete的简单封装.
malloc每次申请内存都会有额外的开销，比如cookie上记录了每次申请的内存的大小。

![图片1](https://user-images.githubusercontent.com/72439295/116352880-6b763800-a828-11eb-9c67-42afd708d495.png)

std::alloc内部维护一个链表数组,数组中的每个链表保存某个尺寸的对象,减少了调用malloc的次数,从而减小了malloc带来的额外开销.

![图片2](https://user-images.githubusercontent.com/72439295/116370370-e9443e80-a83c-11eb-97b9-738d93bc225c.png)

容器

STL容器的各实现类关系如下图所示,以缩排形式表示衍生关系(主要是复合关系).于是可以说stack里面有一个deque,或者说set里面有一个rb_tree.

![图片3](https://user-images.githubusercontent.com/72439295/116371085-a040ba00-a83d-11eb-97a1-36e9da8be9ba.png)

容器list

gcc2.9中list如下所示:

![图片4](https://user-images.githubusercontent.com/72439295/116380521-9f605600-a846-11eb-8ec1-8e71c8a006f5.png)

迭代器__list_iterator重载了指针的*,->,++,--等运算符,并定义了iterator_category、value_type、difference_type、pointer和pointer5个关联类型(associated types),这些特征将被STL算法使用.

![图片5](https://user-images.githubusercontent.com/72439295/116410545-77cdb580-a867-11eb-85e1-314af1af76c1.png)

迭代器的5个关联类型在类中均有定义,但是指针类型的关联类型需要根据指针类别进行确定,为了使STL算法同时兼容迭代器和一般指针,就在迭代器(指针)和算法之间加一个中间层萃取器(traits).

迭代器萃取器iterator_traits能够兼容迭代器和一般指针,获取其5个关联类型:iterator_category、value_type、difference_type、pointer和pointer.

![图片6](https://user-images.githubusercontent.com/72439295/116410729-a0ee4600-a867-11eb-982a-2ba7171ef1fc.png)

容器vector

![图片7](https://user-images.githubusercontent.com/72439295/116498919-76d76b00-a8dd-11eb-966e-8a0ceaa56700.png)

容器vector的迭代器start指向第一个元素,迭代器finish指向最后一个元素的下一个元素,这两个迭代器对应begin()和end()的返回值,维持了前闭后开的特性.

vector对使用者是连续的,因此重载了[]运算符.

vector的实现也是连续的,因此使用指针类型做迭代器(即迭代器vector<T>::iterator的实际类型是原生指针T*).

容器deque

容器deque内部是分段连续的,对使用者表现为连续的.
![ETCZOP$0$(@VIT(1)G2YYRX](https://user-images.githubusercontent.com/72439295/117256389-36e42b00-ae7d-11eb-8323-5f7e18505c36.png)
deque也可叫双端队列，容器内部展现为指针，指向每一个buffer，扩充的时候就添加buffer再用指针指向它。

容器queue和stack

![图片1](https://user-images.githubusercontent.com/72439295/117523061-64e67e00-afe9-11eb-9475-ee58cf91e81c.png)
![图片2](https://user-images.githubusercontent.com/72439295/117523063-66b04180-afe9-11eb-950c-2a92c5e0d5c5.png)

容器queue和stack作为deque的适配器(adapter),其内部均默认封装了一个deque作为底层容器,通过该deque执行具体操作.容器queue和stack的元素进出是严格有序的,因此两个容器都不允许遍历,其内部没有定义iterator.

容器rbtree
 
红黑树是一种二叉查找树，但在每个结点上增加一个存储位表示结点的颜色，可以是red或black。红黑树满足以下五个性质：

1. 每个结点或是红色或是黑色；

2. 根结点是黑色；

3. 每个叶结点是黑的；

4. 如果一个结点是红的，则它的两个儿子均是黑色；

5. 每个结点到其子孙结点的所有路径上包含相同数目的黑色结点。

容器rb_tree封装了红黑树,是有序容器,提供了迭代器iterator用以遍历,但不应使用iterator直接改变元素值(虽然编程层面并没有禁止这样做).

rb_tree提供两种插入操作:insert_unique和insert_equal,前者表示节点的key一定在整棵树中独一无二,否则插入失败;后者表示节点的key可重复.

对于rb_tree,定义一个概念:节点的value包括其key和data,这里的data表示一般说法中的value.

![图片3](https://user-images.githubusercontent.com/72439295/117523167-d7eff480-afe9-11eb-91c8-fc4a71e9aad6.png)

容器set和multiset

容器set和multiset以rb_tree为底层容器,因此其中元素是有序的,排序的依据是key.set和multiset元素的value和key一致.

set和multiset提供迭代器iterator用以顺序遍历容器,但无法使用iterator改变元素值,因为set和multiset使用的是内部rb_tree的const_iterator.

set元素的key必須独一无二,因此其insert()调用的是内部rb_tree的insert_unique()方法;multiset元素的key可以重复,因此其insert()调用的是内部rb_tree的insert_equal()方法.

容器map和multimap

容器map和multimap以rb_tree为底层容器,因此其中元素是有序的,排序的依据是key.

map和multimap提供迭代器iterator用以顺序遍历容器.无法使用iterator改变元素的key,但可以用它来改变元素的data,因为map和multimap内部自动将key的类型设为const.

map元素的key必須独一无二,因此其insert()调用的是内部rb_tree的insert_unique()方法;multimap元素的key可以重复,因此其insert()调用的是内部rb_tree的insert_equal()方法.

map容器重载的[]运算符返回对应data的引用

容器hashtable

![图片4](https://user-images.githubusercontent.com/72439295/117541383-38148400-b046-11eb-9f94-5d9ac4081274.png)

hashtable最开始只有53个桶,当元素个数大于桶的个数时,桶的数目扩大为最接近当前桶数两倍的质数,实际上,桶数目的增长顺序被写死在代码里.如上图所示，而在vs2013中桶最初被定义为8个，当元素超过桶时，每次扩充为原来的二倍。

容器unordered_set、unordered_multiset、unordered_map和unordered_multimap

C++11引入的容器unordered_set、unordered_multiset、unordered_map和unordered_multimap更名自gcc2.9的容器hash_set、hash_multiset、hash_map和hash_multimap,其底层封装了hashtable.用法与set、multiset、map和multimap类似.

迭代器对算法的影响

迭代器的iterator_category类型

迭代器的关联类型iterator_category表示迭代器类型,共5种,用类表示:input_iterator_tag、forward_iterator_tag、bidirectional_iterator_tag、random_acess_iterator_tag和output_iterator_tag.其中前面四个是继承关系，他们之间的继承关系如下：

![图片1](https://user-images.githubusercontent.com/72439295/117633043-3c5cb080-b1b0-11eb-958f-d6a1173665a8.png)

之所以使用类而非枚举来表示迭代器类型,是出于一下两个考虑:

1.使用类的继承可以表示不同迭代器类型的从属关系.

2.STL算法可以根据传入的迭代器类型调用不同版本的重载函数.

![图片2](https://user-images.githubusercontent.com/72439295/117634769-d4a76500-b1b1-11eb-8fcc-4ed40ebf8434.png)

容器array、vector、deque对使用者来说是连续空间,是可以跳跃的,其迭代器是random_access_iterator类型.

容器list是双向链表,容器set、map、multiset、multimap本身是有序的,他们的迭代器都可以双向移动,因此是bidirectional_iterator类型.

容器forward_list是单向链表,容器unordered_set、unordered_map、unordered_multiset、unordered_map哈希表中的每个桶都是单向链表.因此其迭代器只能单向移动,因此是forward_iterator类型.

迭代器istream_iterator和ostream_iterator本质上是迭代器,后文会提到这两个类的源码.

STL中的算法copy根据不同的iterator_category和type_traits执行不同的重载函数.

![图片3](https://user-images.githubusercontent.com/72439295/117635191-2a7c0d00-b1b2-11eb-8f7f-00274a7af0aa.png)

iterater traits和type traits对算法的影响

![图片5](https://user-images.githubusercontent.com/72439295/117637779-b98a2480-b1b4-11eb-946e-787da7452654.png)

算法replace、replace_if、replace_copy

算法replace将范围内所有等于old_value的元素都用new_value取代.

算法replace_if将范围内所有满足pred()为true的元素都用new_value取代.

算法replace_copy将范围内所有等于old_value的元素都以new_value放入新区间,不等于old_value的元素直接放入新区间

算法count、count_if

算法count计算范围内等于value的元素个数.

算法count_if计算范围内所有满足pred()为true的元素个数.

它们支持所有容器,但关联型容器(set、map、multiset、multimap、unordered_set、unordered_map、unordered_multiset和unordered_map)含有更高效的count方法,不应使用STL中的count函数.

算法find、find_if

算法find查找范围内第一个等于value的元素.

算法find_if查找范围内第一个满足pred()为true的元素.

它们支持所有容器,但关联型容器(set、map、multiset、multimap、unordered_set、unordered_map、unordered_multiset和unordered_map)含有更高效的find方法,不应使用STL中的find函数.

算法sort

算法sort暗示参数为random_access_iterator_tag类型迭代器,因此该算法只支持容器array、vector和deque.

容器list和forward_list含有sort方法.

容器set、map、multiset、multimap本身是有序的,容器unordered_set、unordered_map、unordered_multiset和unordered_map本身是无序的,不需要排序.

算法binary_search

算法binary_search从排好序的区间内查找元素value,支持所有可排序的容器.

算法binary_search内部调用了算法lower_bound,使用二分查找方式查询元素.

算法lower_bound和upper_bound分别返回对应元素的第一个和最后一个可插入位置.

迭代器适配器

逆向迭代器reverse_iterator

容器的rbegin()和rend()方法返回逆向迭代器reverse_iterator,逆向迭代器的方向与原始迭代器相反.逆向迭代器适配器reverse_iterator与正常迭代器的方向正好相反:逆向迭代器的尾(头)就是正向迭代器的头(尾);逆向迭代器的加(减)运算就是正向迭代器的减(加)运算.因此逆向迭代器取值时取得是迭代器前面一格元素的值.

![图片1](https://user-images.githubusercontent.com/72439295/117824426-119e5500-b2a1-11eb-9955-0ae9c7f71018.png)

用于插入的迭代器insert_iterator及其辅助函数inserter

迭代器适配器insert_iterator生成用于原地插入运算的迭代器,使用insert_iterator迭代器插入元素时,就将原有位置的元素向后推.insert_iterator通过重载运算符=实现此功能:

![图片2](https://user-images.githubusercontent.com/72439295/117829089-02210b00-b2a5-11eb-9135-ff9bd04155b2.png)

一个万用的Hash Function

之前已经学过的hashtable里面有hash function，在hashtable的上层有unorder_map、unordered_set...,他们都有各自的hash function,整数的hash function是它自己，字符串的hash function是一串递归——当前字符的ASSIC值乘5加下一个字符的ASSIC值的和再乘5... 当我们需要给自己的结构体写一个hash function时，它并不是简简单单的各个元素的hashfunction相加，为了得到更加凌乱的hash code采用下面的经验办法：

![图片1](https://user-images.githubusercontent.com/72439295/117951673-f9811100-b346-11eb-8e28-301056ba927f.png)

容器tuple

容器tuple的源码使用可变模板参数,递归调用不同模板参数的tuple构造函数,以处理任意多的元素类型.因此它的构造过程中产生的中间容器是继承关系.

调用head函数返回的是元素m_head的值.

调用tail函数返回父类成分的起点,通过强制转换将当前tuple转换为父类tuple,丢弃了元素m_head所占内存.

![图片2](https://user-images.githubusercontent.com/72439295/117952348-ad829c00-b347-11eb-900e-ca24ac1b5443.png)

