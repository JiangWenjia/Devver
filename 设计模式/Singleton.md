```objc
@interface Singleton : NSObject
//也可以使用 NS_UNAVAILABLE 标示禁止调用new等方法获取单例实例
+ (Singleton *)sharedInstance;

@end
```

```objc
static Singleton *_sharedSingleton = nil;

@implementation Singleton

- (instancetype)init {
    
    [NSException raise:@"SingletonPattern"
                format:@"Cannot instantiate singleton using init method, sharedInstance must be used."];
    
    return nil;
}

- (id)copyWithZone:(NSZone *)zone {

    [NSException raise:@"SingletonPattern"
                format:@"Cannot copy singleton using copy method, sharedInstance must be used."];
    
    return nil;
}

+ (Singleton *)sharedInstance {

    if (self != [Singleton class]) { //防止子类调用
        
        [NSException raise:@"SingletonPattern"
                    format:@"Cannot use sharedInstance method from subclass."];
    }
    
    static dispatch_once_t onceToken;
    dispatch_once(&onceToken, ^{
        
        _sharedSingleton = [[Singleton alloc] _initInstance];
    });

    return _sharedSingleton;
}

- (id)_initInstance {  //调用父类的init 

    return [super init];
}

@end

```


 

