---
title: "Knowing Your Smart Pointers"
date: 2018-04-25T16:05:16+08:00
url : "/fluentcpp/knowing-your-smart-pointers"
aliases:
    - /post/knowing-your-smart-pointers
type: "post"
author: Jonathan Boccara
draft: false
categories:
  - fluentcpp
tags:
  - fluentcpp
---

# 聪明的开发者使用智能指针（2/7） - 了解 智能指针

*原文链接： https://www.fluentcpp.com/2017/08/25/knowing-your-smart-pointers/*

这是 *聪明的开发者使用智能指针* 系列的第 2 篇，这系列文章包含：

- [智能指针基础](https://www.notfound.me/fluentcpp/smart-developers-use-smart-pointers-smart-pointers-basics/)
- [unique_ptr, shared_ptr, weak_ptr, scoped_ptr, raw_pointers: 了解 智能指针](https://www.notfound.me/fluentcpp/knowing-your-smart-pointers)
- [自定义删除](#) 和 [怎么使它们更加易读](#)
- [在unique_ptr的生命周期中修改其删除行为](#)
- [用unique_ptr实现 PIMPL(Pointer to IMPLementation)](#)
- [在现代C++中实现多态克隆](#)

就像之前我们讨论的,当复制智能指针的时候必须需要一些特殊的操作。否则，默认的拷贝构造方法将会是 未定义行为。

我们有多种合法的操作方式，这些不同的方法引出了不同类型的智能指针。因此一个重要的事情是我们必须清楚这些智能指针的不同，这样我们才能在代码中正确使用它们，并且会使我们的代码具有也更高的可读性。
<!--more-->

下面根据有用性（对于我来说的）列出了一些智能指针：

- std::unique_ptr
- raw pointer
- std::shared_ptr
- std::weak_ptr
- boost::scoped_ptr
- std::auto_ptr

## std::unique_ptr

这个是默认被使用的智能指针，在 C++11 中被引入。

`std::unique_ptr` 的语法意思就是相关的内存资源有且仅有一个拥有者。一个 `std::unique_ptr` 将会持有一个指针，并且在其析构函数中将指针删除（除非你自定义它，这就是另外一个文章的话题了）。

这使你可以使用一个接口来表达你的想法，考虑下面的例子：

```c++
std::unique_ptr<House> buildAHouse();
```

这个接口将会给你一个指向 house 的指针，并且你是这个指针的拥有者。 除了这个 `unique_ptr` 对象以外，**没有其他任何人将会删除这个指针**。并且因为你拥有这个指针，这让你有信心去修改这个指针指向对象的值。注意一个**工厂**类优先使用的返回值就是 `std::unique_ptr`。 事实上，从内存管理方面来说， `std::unique_ptr` 包装了一个普通指针，它是兼容多态性的。

但是这个也适用于另外一种形式，可以把 `std::unique_ptr` 作为一个参数：

```c++
class House
{
public:
    House(std::unique_ptr<PileOfWood> wood);
    ...
```

在这个例子中， house 会取得 `PileOfWood` 的所有权。

注意即使是你接受到一个 `unique_ptr`， 你也不能保证其它人没有访问这个指针的权限。事实上，如果另外一个环境拥有 `unique_ptr` 底层指针的拷贝，你通过 `unique_ptr` 对底层指针的修改也会影响到另外那个环境。但是因为你是这个指针的拥有者，你可以安全合法的修改其指向的对象，其它环境的设计需要考虑到这个影响。如果你不想发生这样的情况，你可以用 `const` 来限定这个 `unique_ptr`：

 ```c++
std::unique_ptr<const House> buildAHouse(); // 因为某些原因，
                                            // 我不希望你修改传给你的底层指针

```

为了确保一个内存资源只被一个 `unique_ptr` 对象拥有， `std::unique_ptr` 对象不能被拷贝。拥有权可以被从一个 `unique_ptr` 使用 `move` 方式 **转移** 到另外一个 `unique_ptr`（最为参数传给一个函数或者从一个函数返回）。 

转移可以从一个函数返回一个 `std::unique_ptr` 的值来实现，或者显式调用：

  ```c++
std::unique_ptr<int> p1 = std::make_unique(42);
std::unique_ptr<int> p2 = move(p1); // 现在p2拥有那个内存资源
                                    //   p1不拥有任何东西
```

## 原始指针 （raw pointers）

什么？你肯定在想我们在讨论 智能指针， 为什么这里会有 原始指针？

即使 原始指针  不是智能指针，但是它们也不是没有存在感的指针。事实上，即使这种情况不是经常发生，我们也有非常合法的理由来使用它们。它们和 引用 有很多共同点，但是后者应该仅仅在某些情况下使用（这是另外一篇文章的话题了）。

现在我只想关注什么是 原始指针和引用： **原始指针和引用代表一个对象的访问权，而不是拥有权**。事实上，这是给函数或方法传输一个对象的默认做法：

```c++
void renderHouse(House const& house);
```

当你希望把一个 `unique_ptr` 对象传递给一个接口的时候这尤为重要。你没有传递这个 `unique_ptr`， 也不是它的引用，而是它指向对象的指针。

## std::shared_ptr

`shared_ptr` 在 C++11时被加入标准，但在 boost 中出现得更早。

**一个内存资源可以同时被多个 `std::shared_ptr` 对象拥有**。`shared_ptr` 在内部维护一个计数器，用来统计有多少对象指向当前内存资源，当最后一个对象被销毁的时候就会删除其对应的内存资源。

第一眼看去，`std::shared_ptr` 像是内存管理的万金油，它可以被传递并且还能维护内存安全。

但是 **`std::shared_ptr` 不应该默认被使用**，这里有一些原因：

- 对于一个资源拥有多个拥有者比只有一个拥有者(像是`unique_ptr`)让系统变得更加**复杂**。即使被 `std::unique_ptr` 拥有的资源不能避免被其他人访问和修改，但是它可以传递一个信息，告诉别人它才拥有其指向资源的特权。基于这个原因，你应该希望从某种程度上统一管理资源。
- 对一个资源有多个拥有者让 **线程安全** 变得更困难。
- 当一个对象没有在某些域(domain)中共享时，它会让代码变得**反常**，让人认为它是“共享”的。
- 因为需要保存它们内部计数器信息，所以它们在时间和内存上是有 **性能** 代价的。

当一个资源在一个域中共享，那么使用 `std::shared_ptr`是一个不错的选择。使用 共享 指针可以提高代码的可读性。一般来讲，图中的节点可以很好的被 共享 指针表示，因为其他节点会保存有一些节点的引用。

![shared_ptr](https://www.fluentcpp.com/wp-content/uploads/2017/04/shared_ptr_graph2-260x300.png)

## std::weak_ptr

`weak_ptr` 在 C++11时被加入标准，但在 boost 中出现得更早。

`std::weak_ptr` 可以同时和 `std::shared_ptr` 保存对一个共享对象的引用，但是它们不会增加引用计数。这意味着如果没有任何 `std::shared_ptr` 指向一个对象，这个对象就会被回收，即使还有一些 `weak_ptr` 指向它。

基于这个原因，一个 `weak_ptr` 需要检查它指向的对象是否存活。为了实现这个，它需要被拷贝进 `std::shared_ptr`:

```c++
void useMyWeakPointer(std::weak_ptr<int> wp)
{
    if (std::shared_ptr<int> sp = wp.lock())
    {
        // 资源还可以被使用
    }
    else
    {
        // 资源已经被删除，不能继续使用
    }
}
```

一个典型的用法是用来 **打破 `shared_ptr` 的循环依赖**。考虑下面的例子：

```c++
struct House
{
    std::shared_ptr<House> neighbour;
};
 
std::shared_ptr<House> house1 = std::make_shared<House>();
std::shared_ptr<House> house2 = std::make_shared<House>();;
house1->neighbour = house2;
house2->neighbour = house1;
```

没有一个 *hourse* 对象会在最后被删除，因为这两个 `shared_ptr` 对象互相指向，形成引用循环。 但是如果其中一个是 `weak_ptr`，那么这个引用循环将会被打破。

另外一个事例在 [Stack Overflow 的这个回答](http://stackoverflow.com/a/106614/6182257) 中。`weak_ptr` 可以用来维护一个 缓存(cache)，其指向的数据或许已经被从缓存中清除。


## boost::scoped_ptr

`scoped_ptr` 出现在 boost 中，但是没有包含在标准库中。

它禁止了 拷贝 和 移动构造方法。 所以它也是唯一一个指向一个资源的对象。它的拥有权也不能被转移。因此 `scoped_ptr` 只能在一个作用域中存活，或者可以作为一个对象的成员。当然，作为一个智能指针，当它被销毁的时候也会删除其指向的资源。

## std::auto_ptr

`auto_ptr` 出现在 C++98 中，在 C++11 中被弃用，并且将会在 C++17 中移除。

它当时被用来提供 `unique_ptr` 的功能，但是那是在 C++ 中不存在 移动(move) 语法。 它用拷贝构造方法中实现了 `unique_ptr` 在 移动构造中实现的功能。但是当你可以使用 `unique_ptr` 时请不要再继续使用 `auto_ptr`，因为它会影响到代码的正确性：

 ```c++
std::auto_ptr<int> p1(new int(42));
std::auto_ptr<int> p2 = p1; // it looks like p2 == p1, but no!
                             //  p1 is now empty and p2 uses the resource

```

你知道 安徒生的丑小鸭吗，一只可怜的小鸭子因为外表不好看被它的兄弟姐妹们嫌弃， 然后等它长大后变成了一只漂亮的白天鹅？ `std::auto_ptr` 的故事也像这个，但是应该把时间倒过来： `std::auto_ptr` 一开始是被用来处理所有权的，但是现在看来在它的同僚（其他智能指针）中显得很糟糕。如果你喜欢的话，可以叫做它是 本杰明 巴顿（译者注：电影《返老还童》的主人公） 的丑小鸭。

🙂

保持关注，下一篇你将会看待如何使用 `std::unique_ptr` 的高级特性来简化内存管理。


