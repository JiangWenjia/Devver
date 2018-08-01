


## valueOfKey
`[obj valueForKey:@"name"];` 通过key或获取值，如果是int等值类型会直接返回NSNumber

查找的步骤：

1. get`<Key>` `<key>` `<isKey>` 的顺序查找方法，有会调用
2. 步骤1没有的话， 会尝试NSArray，NSSet获取元素的方法，获取代理集合
3. 步骤2没有的话，调用 `accessInstanceVariablesDirectly` 返回YES的话（默认YES），会解析实例变量顺序是 `_<key>`, `_is<Key>`, `<key>,` `is<Key>`
4. 步骤三3如果都没有找到，会调用 `valueForUndefinedKey`，不实现则会抛出异常


```objc

#import "KVCObject.h"

@implementation KVCObject {
    NSString * _name;
    NSString * _isName;
    NSString * name;
    NSString * isName;
}

/*--------------------- 1-----------------------------

 顺序 get<Key>,<key> <isKey>  的方法调用
 
 */

- (NSString *)getName {
    // 1-1:-[KVCObject getName
    return [NSString stringWithFormat:@"%s", __func__];
}

- (NSString *)name {
    // 1-2: -[KVCObject name]
    return [NSString stringWithFormat:@"%s", __func__];
}

- (NSString *)isName {
    //1-3: -[KVCObject isName]
     return [NSString stringWithFormat:@"%s", __func__];
}


/*--------------------- 2 -----------------------------
 
1. countOf<Key>
2. 如果有 - (id)object<Key>AtIndex:(NSInteger)index
3. 如果没有，调用 -(NSArray<valueType *> *)<key>AtIndexes:(NSIndexSet *)indexes
 
 
 
最后结果是NSKeyValueArray包含被解析的对象类（说明是使用runtime😓），key 还有解析方法类NSKeyValueNonmutatingArrayMethodSet
 
 */
- (NSInteger )countOfName {
    //2-1: [KVCObject countOfName]
    NSLog(@"%s",__func__);
    return 2;
}


-(id)objectInNameAtIndex:(NSInteger)index {
    //2-2 [KVCObject objectInNameAtIndex:]-0
    NSLog(@"%@",[NSString stringWithFormat:@"%s-%ld", __func__,index]);
    return [NSString stringWithFormat:@"%s-%ld", __func__,index];
}


- (id)nameAtIndexes:(NSIndexSet *)indexes {
    // 2-3 NSKeyValueArray
    NSLog(@"%s",__func__);
    return @[[NSString stringWithFormat:@"nameAtIndexs-%ld",indexes.firstIndex]];
}

/*--------------------- 3 -----------------------------

 
 NSArray                 NSSet                      NSOrderedSet
 
 -countOf<Key>        -countOf<Key>                  -countOf<Key>
                      -enumeratorOf<Key>              -indexIn<Key>OfObject:
 以下两者二选一          -memberOf<Key>:
 -objectIn<Key>AtIndex:                           以下两者二选一
 -<key>AtIndexes:                                 -objectIn<Key>AtIndex:
                                                  -<key>AtIndexes:
 可选（增强性能）
 -get<Key>:range:                                 可选（增强性能）
                                                  -get<Key>:range:
 
 
 
  如果上述没有，accessInstanceVariablesDirectly 返回YES的话（默认YES），会解析实例变量
 按 _<key>, _is<Key>, <key>, is<Key>的顺序访问
 
 */

+ (BOOL)accessInstanceVariablesDirectly {
    NSLog(@"accessInstanceVariablesDirectly");
    return YES;
}



- (instancetype)init {
    self = super.init;
    if(self) {
        _name = @"v_name"; // 3-1
        _isName = @"v_isName"; // 3-2
        name = @"vname"; // 3-3
        isName = @"visName"; //3-4
    }
    return self;
}

/*--------------------- 4 -----------------------------*/
// 4-1 如果都没有找到，会调用 valueForUndefinedKey，不实现则会抛出异常
- (id)valueForUndefinedKey:(NSString *)key {
    NSLog(@"valueForUndefinedKey:%@",key);
    return nil;
}

@end
```


## setValue:forKey：

1. 调用`set<Key>` 方法
2. 没有`set<Key>`方法会调用 `accessInstanceVariablesDirectly`
3. `accessInstanceVariablesDirectly` 返回YES的话，会按照 _<key>, `_is<Key>` ,`<key>`, `is<Key>` 的顺序查找实例变量
4. 查找实例变量未找到调用`setValue：forUndefinedKey：` 方法，默认实现是异常

```objc
//1 调用set<Key> 方法
- (void)setName:(NSString *)name {
    NSLog(@"%s",__func__);
}

// 2.没有set<Key>方法会调用 accessInstanceVariablesDirectly
+ (BOOL)accessInstanceVariablesDirectly {
    NSLog(@"accessInstanceVariablesDirectly");
    return YES;
}

// 3.accessInstanceVariablesDirectly 返回YES的话，会按照 _<key>, _is<Key> ,<key>, is<Key> 的顺序查找实例变量
- (instancetype)init {
    self = super.init;
    if(self) {
        _name = @"v_name"; // 3-1
        _isName = @"v_isName"; // 3-2
        name = @"vname"; // 3-3
        isName = @"visName"; //3-4
    }
    return self;
}

// 4.查找实例变量未找到调用setValue：forUndefinedKey： 方法，默认实现是异常
- (void)setValue:(id)value forUndefinedKey:(NSString *)key {
    NSLog(@"%s",__func__);
}
```

## keyPath

keyPath 使用点语法
`[people valueForKeyPath:@"address.country"]` 获取值
`[people setValue:@(12) forKeyPath:@"address.floor"]` 设置值


```objc
@interface Address : NSObject {
    NSString *_country;
}

@end
@interface Address()
@property (nonatomic, assign) CGPoint floor;
@end
@implementation Address
@end
@interface People : NSObject
@end
@interface People()
@property (nonatomic,copy) NSString* name;
@property (nonatomic,strong) Address* address;
@property (nonatomic,assign) NSInteger age;
@end
@implementation People
@end

//使用
{
    People *people = [People new];
    people.name = @"name";
    people.age  = 19;
    people.address = [Address new];
    [people setValue:@"country" forKeyPath:@"address.country"];
    NSLog(@"%@",[people valueForKeyPath:@"address.country"]);
    [people setValue:@(12) forKeyPath:@"address.floor"];
    NSLog(@"%@",((NSObject *)[people valueForKeyPath:@"address.floor"]).class); // __NSCFNumber
    
}

```

# 正确性的验证

```objc
@interface Address()
@property (nonatomic, assign) CGPoint floor;
@end
@implementation Address


//实现了这个方法的话，验证结果以这个方法为结果，但是之后会调用：validate<Key>:error:
- (BOOL)validateValue:(inout id  _Nullable __autoreleasing *)ioValue forKeyPath:(NSString *)inKeyPath error:(out NSError * _Nullable __autoreleasing *)outError {
    *outError = [NSError errorWithDomain:@"tt" code:1 userInfo:nil];
    NSLog(@"super:%d",[super validateValue:ioValue forKeyPath:inKeyPath error:outError]);
    return NO;
}

// 实现 -(BOOL)validate<Key>:error: kvc会调用次方法来判断 不实现validateValue:的话，这个方法就会作为结果
-(BOOL)validateFloor:(id *)value error:(out NSError * _Nullable __autoreleasing *)outError{
   
    *outError = [NSError errorWithDomain:@"xx" code:1 userInfo:nil];
    return YES;
}

@end

//使用

    NSError  *error;
    NSString *value = @"xaa-a👌";
    if([people validateValue:&value forKeyPath:@"address.floor" error:&error]) { //验证通过
        
    }else { //验证失败
        NSLog(@"%@",error);
    }
    

```

## 字典

```objc
@implementation People

- (void)setValue:(id)value forUndefinedKey:(NSString *)key {
    NSLog(@"------%@",key); // key 不存在会调用这个方法，没实现会抛出异常
}

- (id)valueForUndefinedKey:(NSString *)key {
    NSLog(@"valueForUndefinedKey:-%@",key);
    return @33;
}

@end

//使用
    NSDictionary *dictionary = @{@"name":@"dName", @"age": @1, @"address": address, @"不存在的key":@1};
    
    //字典转模型  key如果不存在会调用 setValue:forUndefinedKey:
    [people setValuesForKeysWithDictionary:dictionary];
    
    //模型转字典（必须使用key数组） key如果不存在会调用valueForUndefinedKey： 并取回返回值作为结果
    NSDictionary *dict = [people dictionaryWithValuesForKeys:@[@"address",@"name",@"不存在的key"]];

```

## 集合操作
1.`valueForKey`，或者`valueForKeyPath` 会作用于每个元素，重新返回一个数组

2.一些集合操作 `@sum`,`@min`,`@max`,`@avg`,`@count`

```
    NSArray *array = @[@"aaaa",@"bbbb",@"ccccc"];
    NSArray *changeArray = [array valueForKey:@"capitalizedString"];
    /*     (
    Aaaa,
    Bbbb,
    Ccccc
    )*/
    NSLog(@"%@",changeArray);
    NSArray *lengthArray = [array valueForKey:@"length"]; // 会作用于每个对象，并且将结果返回组成一个新的数组
    NSLog(@"%@",lengthArray); //(4,4,5)
    
    NSArray *nums = @[@1,@2,@3];
    
    NSLog(@"%@ %@ %@ %@ %@", [nums valueForKeyPath:@"@sum.self"],[nums valueForKeyPath:@"@min.self"],[nums valueForKeyPath:@"@max.self"],[nums valueForKeyPath:@"@avg.self"],[nums valueForKeyPath:@"@count.self"]);
```

## 对象

`@distinctUnionOfObjects`: 去掉集合中的重复元素，并返回新的数组
`@unionOfObjects` ： 获取所有的元素，并返回新的数组

//集合中的集合，没有实验，暂作一下笔记
`@distinctUnionOfArrays`
`@unionOfArrays`
`@distinctUnionOfSets`

```
    // @distinctUnionOfObjects去重，
    NSArray *array =  [@[book1, book2, book3, book4] valueForKeyPath:@"@distinctUnionOfObjects.price"]; //self的话，需要重写equal 和 hash
    NSLog(@"%@",array); // 返回的是价格的数据
```