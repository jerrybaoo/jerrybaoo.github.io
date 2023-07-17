---
layout: post
title:  "区块链存储回收"
date:   2023-07-17 02:14:18 +0800
categories: blockchain 
---
当我们在设计区块链存储时，有时发现某个存储项可能以后在也用不到了。那么此时最好将其删除，以免浪费宝贵的链上存储空间。问题的关键是如何判断某个存储项目是否还有被用到的机会。  
可以参考编程语言当中的垃圾回收系统，当法相某个内存不再被引用时回收它。那么典型的GC 系统显然是不太适合区块链的。那么参考 C++ 或者 Rust 当中的智能指针就比较合适了，它们的核心是引用计数。当智能指针的引用计数为0时释放内存。特别是区块链状态机都是串行执行事务，不要考虑使用原子计数。
那么如果区块链存储项目自己维持一个引用计数，当引用计数为0时删除存储项。这样通过最小的开销，就能完成这个需求。
尤其是当下大部分语言都支持泛型的情况下，那么设计回收方案就更简单了。  
{% highlight golang %}  
    type struct storageA{
        ...
        referenceCount uint64
    }

    type autoGC interface{
        decreaseReference()
        increaseReference()
        ReferenceCount()
        Key()
    } 

    impl autoGC for storageA{
        ...
    }

    type struct GcAction[T autoGC]{
       Storage T    
    }

    func (g GcAction) TryRelease(s store){
        if g.Storage.ReferenceCount() == 0{
            s.delete(g.Storage.Key())
        }
    }
{% endhighlight %}


