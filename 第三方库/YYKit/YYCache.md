# YYMemoryCache
使用LRU（least-recently-used）


## 数据结构
_YYLinkedMapNode

### 链表节点 `_YYLinkedMapNode`
```objc
    __unsafe_unretained _YYLinkedMapNode *_prev; // 前一个节点
    __unsafe_unretained _YYLinkedMapNode *_next; //下一节点
    id _key;
    id _value;
    NSUInteger _cost;
    NSTimeInterval _time;
```
### 链表`_YYLinkedMap`
双向链表
```objc
    CFMutableDictionaryRef _dic; // do not set object directly
    NSUInteger _totalCost;
    NSUInteger _totalCount;
    _YYLinkedMapNode *_head; // MRU, do not change it directly
    _YYLinkedMapNode *_tail; // LRU, do not change it directly
```
方法

```objc
/// Insert a node at head and update the total cost.
/// Node and node.key should not be nil.
- (void)insertNodeAtHead:(_YYLinkedMapNode *)node;

/// Bring a inner node to header.
/// Node should already inside the dic.
- (void)bringNodeToHead:(_YYLinkedMapNode *)node;

/// Remove a inner node and update the total cost.
/// Node should already inside the dic.
- (void)removeNode:(_YYLinkedMapNode *)node;

/// Remove tail node if exist.
- (_YYLinkedMapNode *)removeTailNode;

/// Remove all node in background queue.
- (void)removeAll;
//把dict重新建立了一个，然后在release 了 holder

```

### 关于CFMutableDictionaryRef的使用

```objc
//声明
CFMutableDictionaryRef _dic; // do not set object directly
//创建
_dic = CFDictionaryCreateMutable(CFAllocatorGetDefault(), 0, &kCFTypeDictionaryKeyCallBacks, &kCFTypeDictionaryValueCallBacks);
//设置值
CFDictionarySetValue(_dic, (__bridge const void *)(node->_key), (__bridge const void *)(node));
//删除值
CFDictionaryRemoveValue(_dic, (__bridge const void *)(_tail->_key));
//移除
CFRelease(holder);

```

# `YYMemoryCache`

释放对象的时候，先从链表中移除。并添加的局部数组中，在对应线程中对应的线程中释放

```objc
    NSMutableArray *holder = [NSMutableArray new];
    while (!finish) {
        if (pthread_mutex_trylock(&_lock) == 0) {
            if (_lru->_tail && (now - _lru->_tail->_time) > ageLimit) {
                _YYLinkedMapNode *node = [_lru removeTailNode];
                if (node) [holder addObject:node];
            } else {
                finish = YES;
            }
            pthread_mutex_unlock(&_lock);
        } else {
            usleep(10 * 1000); //10 ms
        }
    }
    if (holder.count) {
        dispatch_queue_t queue = _lru->_releaseOnMainThread ? dispatch_get_main_queue() : YYMemoryCacheGetReleaseQueue();
        dispatch_async(queue, ^{
            [holder count]; // release in queue
        });
    }
```

# YYDiskCache

//主要可以使用弱应用的方式，保存
[NSHashTable](https://www.jianshu.com/p/de71385930ba)
// ???: NSMapTable





