# Runtime

>  参考资料

[杨萧玉-Objective-C Runtime](http://yulingtianxia.com/blog/2014/11/05/objective-c-runtime)


# Runtime 基础数据结构

```objc
id objc_msgSend ( id self, SEL op, ... );
```

## SEL

映射方法的C字符串、区分方法的ID，数据结构就是SEL

```objc
typedef struct objc_selector *SEL;
```

不同的类的方法选择器是相同的，方法名字相同变量类型不同方法选择器也是相同的。

可以使用@selector() 或者 sel_registerName 函数获取

## id

实例类型的指针

```objc
typedef struct objc_object *id;

```

```objc
struct objc_object {
private:
    isa_t isa;
}
```

isa指针，类型是isa_t 联合体，不总是指向所属的类，所以应该使用class方法确定对象的类，例如KVO使用**isa-swizzling**的技术，把isa指向中间类而非真实的类


## class

```objc

typedef struct objc_class *Class;

struct objc_class : objc_object {
    // Class ISA;
    Class superclass;
    cache_t cache;             // formerly cache pointer and vtable
    class_data_bits_t bits;    // class_rw_t * plus custom rr/alloc flags
    class_rw_t *data() { 
        return bits.data();
    }
    ... 省略其他方法
}

```

objc_class 继承自 objc_object类，ObjC类本身类对象也是一个对象，他们是Meta class（元类）类型的实例变量。每个类对象仅有一个与之对应的元类。发送 ==[NSObject alloc]== 消息时发送给NSObject 类的实例变量。

![image](http://7ni3rk.com1.z0.glb.clouddn.com/Runtime/class-diagram.jpg)

- 类对象的实例isa指向类对象，类对象的isa指向元类对象
- 所有的元类对象的isa指向根元类
- 根元类的isa指向自己，超类指向NSObject
- NSObject的超类为nil
- 

## class_data_bits_t 

### class_ro_t
存储编译器就确定的信息，存储类的方法、属性、协议、成员变量

class_data_bist_t 中包含了 class_rw_t 而class_rw_t中包含了class_ro_t


```objc
struct class_ro_t {
    uint32_t flags;
    uint32_t instanceStart;
    uint32_t instanceSize;
#ifdef __LP64__
    uint32_t reserved;
#endif

    const uint8_t * ivarLayout;
    
    const char * name;
    method_list_t * baseMethodList;
    protocol_list_t * baseProtocols;
    const ivar_list_t * ivars;

    const uint8_t * weakIvarLayout;
    property_list_t *baseProperties;

    method_list_t *baseMethods() const {
        return baseMethodList;
    }
};
```

### class_rw_t
提供运行时对类的扩展能力，二者都存储着类的方法、属性（成员变量）、协议

```objc
struct class_rw_t {
    // Be warned that Symbolication knows the layout of this structure.
    uint32_t flags;
    uint32_t version;

    const class_ro_t *ro;

    method_array_t methods;
    property_array_t properties;
    protocol_array_t protocols;

    Class firstSubclass;
    Class nextSiblingClass;

    char *demangledName;

#if SUPPORT_INDEXED_ISA
    uint32_t index;
#endif
	... 省略操作 flags 的相关方法
}
```

### realizeClass
在类初始化之前返回都是calss_ro_t 等到 realizeClass调用时，开辟class_rw_t 空间，将calss_ro_t 赋值给calss_rw_t -> ro


## 分类

在加载镜像时候调用attachCategories 函数，会在calss_rw_t中的==method_array_t==, ==property_array_t==, ==protocol_array_t== 会添加==method_list_t==, ==property_list_t==, ==protocol_list_t==指针


## Method

```objc
typedef struct method_t *Method;

struct method_t {
    SEL name;
    const char *types;
    IMP imp;

    struct SortBySELAddress :
        public std::binary_function<const method_t&,
                                    const method_t&, bool>
    {
        bool operator() (const method_t& lhs,
                         const method_t& rhs)
        { return lhs.name < rhs.name; }
    };
};

```

- SEL 方法名称
- types 方法的参数类型和返回值
- IMP 指向一个方法的实现函数


## 动态方法解析

==@dynamic== 修饰之后，不会生成setter和getter，需要动态提供，通过重载 resolveInstanceMethod: 和 resolveClassMethod: 方法分别添加实例方法和类方法的实现。

```
@dynamic propertyName;

void dynamicMethodIMP(id self, SEL _cmd) {
    // implementation ....
}
@implementation MyClass
+ (BOOL)resolveInstanceMethod:(SEL)aSEL
{
    if (aSEL == @selector(resolveThisMethodDynamically)) {
          class_addMethod([self class], aSEL, (IMP) dynamicMethodIMP, "v@:");
          return YES;
    }
    return [super resolveInstanceMethod:aSEL];
}
@end
```
v：void @：id类型 ：表示SEL


# 消息转发

![image](http://7ni3rk.com1.z0.glb.clouddn.com/QQ20141113-1@2x.png?imageView2/2/w/800/q/75|watermark/2/text/eXVsaW5ndGlhbnhpYQ==/font/Y29taWMgc2FucyBtcw==/fontsize/500/fill/I0VGRUZFRg==/dissolve/100/gravity/SouthEast/dx/10/dy/10)





## 重定向
替换方法的接收者

```
- (id)forwardingTargetForSelector:(SEL)aSelector
{
    if(aSelector == @selector(mysteriousMethod:)){
        return alternateObject;
    }
    return [super forwardingTargetForSelector:aSelector];
}
```


## 转发

```
- (void)forwardInvocation:(NSInvocation *)anInvocation
{
    if ([someOtherObject respondsToSelector:
            [anInvocation selector]])
        [anInvocation invokeWithTarget:someOtherObject];
    else
        [super forwardInvocation:anInvocation];
}


```




