---
title: "Ways Reordering Collection Stl"
date: 2018-04-20T22:10:40+08:00
url : "/fluentcpp/ways-reordering-collection-stl"
aliases:
    - /post/ways-reordering-collection-stl
type: "post"
author: Jonathan Boccara
draft: false
categories:
  - fluentcpp
tags:
  - fluentcpp
---
# STL 中容器重新排列的方法

*本文翻译自：https://www.fluentcpp.com/2018/04/20/ways-reordering-collection-stl/*

STL 库可以对容器做很多操作， 其中一个就是可以对容器中的元素进行重新排序。或者换句话说，可以对容器进行组合排列(premutation)。

事实上，移动容器中的元素一般需要写相当数量的复杂代码，其中需要 循环(loop) 和 迭代(iterators)。 把这些复杂操作封装成有意义的接口或许是 STL 最引人注目的成就。

让我们来看一下 STL 提供的 排列 种类：

- 字典序 排列
- 循环 排列
- 随机 排列
- 逆序
- 检查排列
- 其他排列方式


<!--more-->
## 字典序 排列

一个给定的含有 N 个元素的容器可以有很多种方法重新排列 (准确来说是 N! 种方式)。在所有的排列方式上迭代 (iterate)而不漏掉任何种是可能的吗？

为了实现这个，我们可以在给定的排列集合上定义一个 **顺序(order)**. 这种方式中我们可以从一种排列去到另一种排列，直到我们返回开始的地方。

但是有没有一种自然的排列呢？

结论是有的：我们可以按照 **字典序(lexicographical order)** 排列一个给定的集合。 把每种排列想像成一个”单词“，容器中中的元素作为单词的”字母“。

然后我们就可以把这些单词按照 “字典序” (这里用引号表示这里我们不是讨论真正的 `字母` 和 `字符串`， 仅仅是一个想法)。 为了让这个起作用，我们需要对容器中的元素实现一个 `operator<` 比较方法

下面是一个容器 {1,2,3,4,5} 的四种 按字典序升序 的排列方式：

```
{1, 2, 3, 4, 5}
{1, 2, 3, 5, 4}
{1, 2, 4, 3, 5}
{1, 3, 4, 5, 3}
...
``` 

那么现在我们怎么用 STL 做这些呢？

为了从一个排列到下一个字典序排列，我们使用： `std::next_premutation`：

```c++
vector<int> v = {1, 2, 3, 4, 5 };
 
std::next_permutation(v.begin(), v.end()); // v 现在包含 {1, 2, 3, 5, 4}
```

`std::next_premutation` 返回一个 `bool` 值， 如果得到的排列是字典序大于原排列 (任意一个大于排列) 就返回 true, 否则返回 false (唯一的例子是下一个迭代返回到了之前第一个（最小的）排列方式)。

如果想从一个排列返回之前的排列， 可以使用 `std::prev_premutation`：

```c++
vector<int> v = {1, 2, 3, 5, 4};
 
std::prev_permutation(v.begin(), v.end()); // v now contains {1, 2, 3, 4, 5 }
```

对称的， `std::prev_premutation` 返回一个 `bool` 值， 如果得到的排列是字典序小于原排列 （任意一个小于排列）就返回 true，否则返回 false（唯一的例子时下一个迭代返回到了最后一个（最大的）排列方式）。

`std::next_premutation` 和 `std::prev_premutation` 直接作用在输入参数指定的范围内， 这可以方便的多次调用他们：

```c++
std::vector<int> numbers = {1, 2, 3, 4};
while (std::next_permutation(begin(numbers), end(numbers)))
{
    for (int n : numbers) std::cout << n << ' ';
    std::cout << '\n';
}
```

上面的代码输出：

```bash
1 2 4 3 
1 3 2 4 
1 3 4 2 
1 4 2 3 
1 4 3 2 
2 1 3 4 
2 1 4 3 
2 3 1 4 
2 3 4 1 
2 4 1 3 
2 4 3 1 
3 1 2 4 
3 1 4 2 
3 2 1 4 
3 2 4 1 
3 4 1 2 
3 4 2 1 
4 1 2 3 
4 1 3 2 
4 2 1 3 
4 2 3 1 
4 3 1 2 
4 3 2 1
```

这些是{1, 2, 3, 4, 5} 的所有的排列方式， 下次循环就会回到初始状态。


## 循环 排列
一个 循环排列 把容器里的元素移动容器末尾的元素并且放置到容器开始的位置上。 举个例子， 下面是 {1, 2, 3, 4, 5} 的所有循环排列：

```
{1, 2, 3, 4, 5}
{5, 1, 2, 3, 4}
{4, 5, 1, 2, 3}
{3, 4, 5, 1, 2}
{2, 3, 4, 5, 1}
```

对于一个有 N 个元素的容器来说， 有 N 种不同的 循环排列

### 基本使用方法

在 C++ 里面， 使用 `std::rotate` 来进行循环排列。

`std::rotate` 使用 3 个 迭代器：

    - 一个指向排列范围开始
    - 一个指向你想把它排在最前面的元素
    - 一个指向排列范围的末尾

在 C++11 里面， `std::rotate` 返回一个使指定元素排在第一位的迭代器。下面是它的接口：

```c++
template<typename ForwardIterator>
ForwardIterator rotate(ForwardIterator begin, ForwardIterator new_begin, ForwardIterator end);
```
![std::rotate](https://www.fluentcpp.com/wp-content/uploads/2017/12/rotate.png)

在 C++98 里面这个接口有一点点不同： 返回值为 void：

```c++
template<typename ForwardIterator>
void rotate(ForwardIterator begin, ForwardIterator new_begin, ForwardIterator end);
```

`std::rotate` 直接作用（修改）传入的排列范围内的元素。如果你想保持原范围元素不变，可以使用 `std::rotate_copy` 将输出保存到另外一个容器中。

### 一个 std::rotate 的有趣用法

`std::rotate` 可以用来创建新的算法， Sean Parent 在 GoingNative 2013 上做过的 [C++ Seasoning](https://channel9.msdn.com/Events/GoingNative/2013/Cpp-Seasoning) 中提到过这个事情。让我们来看一下 Sean 当时的例子， 这个例子展示了 STL 的能力。

这个例子如下： 给定一个范围 (range)， 怎么实现一个算法将一个子范围内连续元素滑动到这个范围内指定的位置？

![std::rotate1](https://www.fluentcpp.com/wp-content/uploads/2017/12/rotate1.png)

试着想一下你将如何实现这个算法，大体上想一下这个问题的的复杂度。

事实上，将从 `first` 到 `last` 范围内的元素滑动到 `pos` 的位置上等价于在 `first` 到 `pos`范围上运行一个 把 元素`last`放到最前面的循环排列， 这就是 `std::rotate` 用来做的事情：

```c++
std::rotate(first, last, pos);
```

![std::rotate2](https://www.fluentcpp.com/wp-content/uploads/2017/12/rotate2.png)

现在这个算法只工作在 `last < pos` 的时候， 意思就是这些元素可以向后滑动。 那么怎么样向前滑动到 `pos < first` 的位置呢？

向前滑动元素也是做范围从 `pos` 到 `last` 的循环排列， 但是这次是把 元素 `first` 放到最前面， 所以实现方式是：

```c++
std::rotate(pos, first, last);
```

现在如果 `pos` 是在 `first` 和 `last` 之间，意味着需要滑动的元素已经在它们要在的地方了，我们不需要做任何事情。

把上面的放到一起，实现方式就是：

```c++
if (pos < first) std::rotate(pos, first, last);
if (last < pos) std::rotate(first, last, pos);
```
基于 C++11 的接口，它返回应用 `std::rotate` 之前范围内元素的新位置， 我们可以让它返回应用 `std::rotate` 之后那些元素的范围：

- 如果 `pos < first`， 滑动后的元素位于 `pos` 和 排列后新的第一个元素之间 （不是 滑动区间）， 这是 `std::rotate（pos, first, last）` 的返回值。
- 如果 `last < pos`， 滑动后的元素位于排列后第一个元素和 `pos` 之间

总结一下， 滑动算法如下实现：

```c++
template <typename ForwardIterator>
std::pair<ForwardIterator, ForwardIterator> slide(ForwardIterator first, ForwardIterator last, ForwardIterator pos)
{
    if (pos < first) return { pos, std::rotate(pos, first, last) };
    if (last < pos) return { std::rotate(first, last, pos), pos };
    return { first, last };
}
```

哪怕这个算法跟在容器上做 排列 没有任何联系， 我们可以看到， 在这个例子中，返回一对 迭代器 也是有问题的。事实上，我们需要的是一个真正用它 begin 和 end 表示的范围，

基于以上原因， 我们可以考虑提升一下 [抽象层次](https://www.fluentcpp.com/2016/12/15/respect-levels-of-abstraction/)  并且返回一个更好的可以表示我们想法的类型。 [`boost::iterator_range`](http://www.boost.org/doc/libs/1_54_0/libs/range/doc/html/range/reference/utilities/iterator_range.html) 或者 range-v3 的 [`iterator_range`](https://github.com/ericniebler/range-v3/blob/ca997df10962c482274e6be37fdbe39add8664c9/include/range/v3/iterator_range.hpp) 类 都是这样的想法。注意我们已经在 [find something efficiently with the STL](https://www.fluentcpp.com/2017/01/16/how-to-stdfind-something-efficiently-with-the-stl/) 中查找 `std::equal_range` 接口时遇到这些了。

## 随机 排列

一个简单的重排列一个容器中所有元素的方法是将它们随机排列。

`std::shuffle` 可以用来做这个：

```c++
#include <random>
#include <algorithm>
#include <vector>
 
std::vector<int> numbers = {1, 2, 3, 4, 5, 6, 7, 8, 9, 10};
 
std::random_device randomDevice;
std::mt19937 generator(randomDevice());
 
std::shuffle(begin(numbers), end(numbers), generator);
 
for (int n : numbers) std::cout << n << ' ';
```

上面代码输出一个新的排列：

```
8 10 5 1 7 2 3 6 4 9
```

### 命运注定的（不太好的） `std::random_shuffle`

这里有个重要的事情： 在 C++11 之前有 `std::random_shuffle` 可以实现这个特性，但是它的随机源 (`rand()`) 不是很理想。 所以它将在 `C++14` 里被弃用并且在 `C++17` 中删除。 所以你不应该再继续使用它。

另一方面， C++11里面引入了它的继任者 `std::shuffle`。 那么在 C++98 里面怎么在不引入技术负累的基础上重新排列一个容器呢？

如果你曾经遇到过这个问题（我还没有）， 如果你把可以它分享出来是最好的, 因为现在在 C++ 社区中已经有很少人还在迁移到 C++11 的过程中。

## 逆序 排列

一个更简单的排列方式就是逆序排列一个容器了， 可以使用 `std::reverse`来做到！

```c++
std::vector<int> numbers = {1, 2, 3, 4, 5, 6, 7, 8, 9, 10};
 
std::reverse(begin(numbers), end(numbers));
```

输入如下：

```
10 9 8 7 6 5 4 3 2 1
```

## 排列的检查

假设现在我们有两个容器。你怎么确定一个是另外一个的一种排列呢？ 或者另外一种说法： 是不是一个容器与另外一个容器包含同样的元素(排列方式或许不同)?

举个例子，我们有下面的容器：

```c++
std::vector<int> v1 = {1, 2, 3, 4, 5};
std::vector<int> v2 = {4, 2, 3, 1, 5};
std::vector<int> v3 = {2, 3, 4, 5, 6};
```

这样调用 `std::permutation`:

```c++
std::is_permutation(v1.begin(), v1.end(),
                    v2.begin(), v2.end());
```
返回值是 `true`， 如果：

```c++
std::is_permutation(v1.begin(), v1.end(),
                    v3.begin(), v3.end());
```

返回值是 `false`。因为容器 v3 和 v1 中的元素并不相同。

在 C++14 之前， `std::permutation` 有一个 1.5-Range 的接口，这是说它接受第一个容器的 begin 和 end， 但是它仅仅接受第二个容器的 **begin**：

```c++
std::is_permutation(v1.begin(), v1.end(),
                    v2.begin());
```

所以如果第二个容器小于第一个容器，那么这个算法将越过第二个容器的 end 直到到达第一个容器的 end。这将是一个为定义行为。后果就是你必须保证第二个容器必须大于等于第一个容器的大小。

但是在 C++14 中这将是正确的，它增加了一个重载对两个容器大小的额外检查。

`std::is_permutation` 用 **operator==** 进行元素比较， 并且可以接受自定义的比较函数。

### `std::is_permutation` 的算法复杂度

`std::is_permutation` 具有至多 ”O(n)“ 的复杂度。

这听起来很令人吃惊： 事实上，STL 中的算法都是用可能好的算法复杂度来实现。 并且这个看起来我们可以用更好的平方根的复杂度，是不是呢？

其实是可以的，不过需要付出更多的内存消耗，如果你对这个比较感兴趣，我建议你去看一下 Quentin 的文章 [Lost in Permutation Complexity](https://deque.blog/2017/04/04/lost-in-permutation-complexity/)。 所以这是一个 CPU 和 内存的折中。 听起来很熟悉，是吧？

### `std::is_permutation` 的一个使用例子

试想一下如果一个函数返回一个容器 （或者用 [output iterator](https://www.fluentcpp.com/2017/11/28/output-iterator-adaptors-symmetry-range-adaptors/)来产生，但是没有指定内部元素的位置。

你怎么给这个函数写一个单元测试呢？

你不能对期望的输出和实际输出使用 `EXPECT_EQ`， 我们并不知道实际输出的是什么顺序。

相反的，现在你可以使用 `std::is_permutation`：

```c++
std::vector<int> expected = {1, 2, 3, 4, 5};
 
std::vector<int> results = f();
 
EXPECT_TRUE(std::is_permutation(begin(expected), end(expected),
                                begin(results), end(results)));
``` 

这样你就可以表示你想 `f` 函数以任何顺序输出 1,2,3,4,5。

## 其他的 排列

我们已经说过 STL 中所有的改变容器内元素排列方式的方法了吗？

并没有！ 还有其它类型的排列方式，它们值得去它们自己的文章中深度阅读：

- [Partitioning with the STL algorithms](https://www.fluentcpp.com/2017/10/10/partitioning-with-the-stl/)
- [Sorting with the STL algorithms](https://www.fluentcpp.com/2017/10/03/sorting-stl-algorithms/)
- Operating on Heaps with the STL algorithms