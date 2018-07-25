### RACSignal
> 常用于command的网络请求中的信号

 -  RACSignal: +createSignal 创建出来的属于RACDynamicSignal
 -  订阅一次就会触发一次didSubscribe的block（本身用于创建订阅所以会触发）
 
 - 直接调用RACDisposable 不会立马释放，生命周期同ARC ）complete，error都会触发


```objc
	RACSignal *siganl = [RACSignal createSignal:^RACDisposable *(id<RACSubscriber> subscriber) {
		
		NSLog(@"signal-block触发了");
		[subscriber sendNext:@(2)];
		[subscriber sendCompleted];
		return [RACDisposable disposableWithBlock:^{
			NSLog(@"信号销毁了");
		}];
	}];
	
	
	RACDisposable *dispose =  [siganl subscribeNext:^(id x) {
		NSLog(@"收到信号:%@",x);
	} error:^(NSError *error) {
		NSLog(@"错误%@",error);
	} completed:^{
		NSLog(@"信号发送完毕");
	}];
	
	[dispose dispose]; //调用并不会立马释放
	
	[siganl subscribeNext:^(id x) { //会再次调用创建的block的
		NSLog(@"2：：：：：%@",x);
	}];
	
	@weakify(siganl)
	dispatch_after(dispatch_time(DISPATCH_TIME_NOW, (int64_t)(1 * NSEC_PER_SEC)), dispatch_get_main_queue(), ^{
		@strongify(siganl)
		[siganl subscribeNext:^(id x) {
			NSLog(@"%@",x);
		}];
	});
```

### RACCommand

> 常用于网络请求

三个常用属性

- executionSignals 信号中的信号：command当执行的时候，就会发出信号，信号的值就是信号

- errors command的执行错误，通过订阅这个信号，一般还可以合并一个vm里的多个command的errors信号，统一处理错误

- 

```objc

RACSignal *availableSignal = [self.userNameTextField.rac_textSignal map:^(NSString *value) {
		return value.length > 3 ? @(YES) : @(NO);
	}];
	
	//initWithEnabled 参数传递BOOL类型事件，当为YES的是command才可以用
	RACCommand *command = [[RACCommand alloc] initWithEnabled:availableSignal signalBlock:^RACSignal *(id input) {
		
		NSLog(@"command 到我啦");
		return [RACSignal createSignal:^RACDisposable *(id<RACSubscriber> subscriber) {
			NSLog(@"我可以用");
			[subscriber sendNext:nil];
			[subscriber sendError:nil]; //必须调用结束或者error方法，不然command只会执行一次
			return [RACDisposable disposableWithBlock:^{
				NSLog(@"结束啦");
			}];
		}];
	}];
	
//	self.command  = command;
	
//	@weakify(self)
//	[[self.loginButton rac_signalForControlEvents:UIControlEventTouchUpInside] subscribeNext:^(id x) {
//		NSLog(@"点击啦");
//		@strongify(self)
//		[self.command execute:nil];
//	}];
	
	
	//button 可以绑定一个command 同时 当command 可以用的时候，button的enable才可以用，意思是不能再使用RAC(self.button, enable)
	self.loginButton.rac_command = command;
	
	[command.executionSignals subscribeNext:^(RACSignal *x) {
		// command 开发发出信号（表示singal里block要执行了）
		NSLog(@"command 里signalBlock 要开始发出signal（return回来的信号）");
		[x subscribeNext:^(id x) {
			
			NSLog(@"command 里signal执行啦");
		}];
	}];
	
	
	[[command.executionSignals switchToLatest] subscribeNext:^(id x) {
		 NSLog(@"command 里signal执行啦2");
	} error:^(NSError *error) { //这里是不会执行的
		NSLog(@"command 里signal执行error");
	}];
	
	//订阅 command return里发送的 error  （应该这么订阅）
	[command.errors subscribeNext:^(id x) {
		NSLog(@"command 里signal执行error2");
	}];
```

 ### RACSubject 
  > 常用于代理
 
 - 注意订阅顺序 先发送后订阅不会接收到信号
   其本身是保存所有的订阅者，遍历发送
 
 - RACReplaySubject 后订阅，也会收到之前发送的信号
   其本身会订阅信号，并保存值，遍历所有的订阅者发送值
   
   
```objc
- (void)subject {
	
	RACSubject *subject = [RACSubject subject];
	

	//先发送
	[subject sendNext:@(1)]; //订阅者不会接受到
	
	//注意这里是订阅的位置
	[subject subscribeNext:^(id x) {
		NSLog(@"1：%@",x);
	}];
	
	[subject subscribeNext:^(id x) {
		NSLog(@"2：%@",x);
	}];
	
	//后发送
	[subject sendNext:@(2)]; //两个订阅者都会接收到
	
	
	RACReplaySubject *replaySubject = [RACReplaySubject subject];
	
	
	//先发送，订阅者都会接收到
	[replaySubject sendNext:@(1)];
	[replaySubject sendNext:@(0)];
	
	// 1 信号会接收到 2信号也是
	[replaySubject subscribeNext:^(id x) {
		NSLog(@"replay:1：%@",x);
	}];
	
	
	[replaySubject subscribeNext:^(id x) {
		NSLog(@"replay:2：%@",x);
	}];
	
	[replaySubject sendNext:@(2)];
}
```

### RACMulticastConnection
 > 为了消除 RACSignal 被订阅后block被执行多次
 
 使用方法：
 1. 通过信号 [signal publish]
 2. 订阅信号
 3. 连接信号 [connect connect]


```objc
	RACSignal *signal = [RACSignal createSignal:^RACDisposable *(id<RACSubscriber> subscriber) {
		NSLog(@"发送数据");
		[subscriber sendNext:@(1)];
		return nil;
	}];
	
	RACMulticastConnection *connect = [signal publish];
	// [connect connect]; 会执行signal中的信号，但是订阅者后订阅的接收不到信号
	
	[connect.signal subscribeNext:^(id x) {
		NSLog(@"1");
	}];
	
	[connect.signal subscribeNext:^(id x) {
		NSLog(@"2");
	}];
	
	//注意连接放到了订阅之后
	[connect connect];
```

# 值的操作

### bind 
> 信号转发
 
 信号绑定后对数据进行处理，返回一个新的信号


```
- (void)bind {
	
	RACSubject *subject = [RACSubject subject];
	
	RACSignal *bindSignal  = [subject bind:^RACStreamBindBlock{
		
		return ^ RACStream * (NSNumber *value, BOOL *stop) {
		
			NSLog(@"收到数据:%@",value);
			//处理订阅的信号
			return [RACSignal return:@(value.integerValue+1)];
		};
	}];
	
	
	[bindSignal subscribeNext:^(id x) {
		NSLog(@"订阅者收到数据:%@",x);
	}];
	
	[subject sendNext:@(1)];
}
```

### distinctUntilChanged & map & flattenMap
 
 - distinctUntilChanged: 去重，前后两次一样只会发送一次
 
 - map： 把信号发送的值，隐射成其他值（类型）
 
 - flattenMap： 把值隐射成信号，可以直接订阅


```objc
	RACSubject *subject = [RACSubject subject];
	
	[[[[subject distinctUntilChanged] map:^id(id value) {
		return [NSString stringWithFormat:@"%@", value];
	}] flattenMap:^RACStream *(NSString *value) {
		NSLog(@"%@",value);
		return [RACSignal return:[NSString stringWithFormat:@"flat-%@",value]];
	}] subscribeNext:^(id x) {
		NSLog(@"订阅到:%@",x);
	}];
	
	[subject sendNext:@(1)];
	[subject sendNext:@(1)]; //只会收到一次
	[subject sendNext:@(2)];
```

### reduceEach
 
 > 聚合元组值类型信号,返回聚合值


```objc
- (void)reduce {
	
	RACSignal *signal = [RACSignal return:RACTuplePack(@(3),@(4))];
	[[signal reduceEach:^id(NSNumber *first, NSNumber *second){
		return @(first.integerValue + second.integerValue);
	}] subscribeNext:^(id x) {
		NSLog(@"%@",x);
	}];
}
```

### scan

 > 聚合前一个信号值和下一个信号值


```objc
- (void)scan {
	RACSequence *nums = @[@1, @2, @3].rac_sequence;
	
	[[nums scanWithStart:@4 reduce:^id(NSNumber *running, NSNumber *next) {
		return @(running.integerValue + next.integerValue);
	}].signal subscribeNext:^(id x) {
		NSLog(@"合为 %@", x); // 5 7 10
	} completed:^{
		
	}];
}
```


### filter & ignore & take & takeLast & skip
 
 filter: 过滤为返回false的值
 
 ignore： 忽略某个值
 
 take： 从前往后取多少个值
 
 takeLast :  从后往前取多少个值，信号必须sendCompleted之后，才会收到值
 
 skip： 从前往后跳过多个值


```objc
	RACSubject *subject = [RACSubject subject];
	
	RACSignal *siganl = [[subject filter:^BOOL(NSNumber *value) {
		return value.integerValue > 3;
	}] ignore:@(4)];
	
	[[siganl take:2] subscribeNext:^(id x) {
		NSLog(@"1：%@",x); // 5,6
	}];
	 
	[[siganl takeLast:2] subscribeNext:^(id x) {
		NSLog(@"2: %@",x);// 6 7
	}];
	
	[[subject skip:2] subscribeNext:^(id x) {
		NSLog(@"3: %@",x);//7
	}];
	
	[subject sendNext:@(4)];
	[subject sendNext:@(5)];
	[subject sendNext:@(6)];
	[subject sendNext:@(7)];
	
	[subject sendCompleted];
```

### startWith & repeat
 
 startWith： 会在开始之前多添加一个信号值
 
 repeat： 会一直循环，需要手动取消



```objc
	RACSignal *signal = @[@1, @2, @3, @4].rac_sequence.signal;
	
	[[signal startWith:@9] subscribeNext:^(id x) {
		NSLog(@"%@", x); // 9 1 2 3 4
	}];
	
	RACDisposable *dispose = [[signal repeat] subscribeNext:^(id x) {
		NSLog(@"repeat:%@", x); // 1 2 3 4 ....循环
	}];
	
	dispatch_after(dispatch_time(DISPATCH_TIME_NOW, (int64_t)(1 * NSEC_PER_SEC)), dispatch_get_main_queue(), ^{
		[dispose dispose];
	});
```

### retry
 
 retry ： 如果 sendError 之后，会重试
 
 retry：2 发送error之后会重试2次
 retry    直到成功


```objc
- (void)retry {
	__block int time = 0;
	RACSignal *signal = [RACSignal createSignal:^RACDisposable *(id<RACSubscriber> subscriber) {
		NSLog(@"重试");
		time++;
		if(time > 100) {
			[subscriber sendNext:@1];
		}else {
			[subscriber sendError:nil];
		}
		return nil;
	}];
	
	[[signal retry:2] subscribeNext:^(id x) {
		NSLog(@"2--try: %@",x); // 结果会受到下面一个订阅者的retry的影响，所以要分开测试，因为time也会被加
	} error:^(NSError *error) {
		NSLog(@"2--try -error"); //重试两次（3次执行）没有成功后会发送失败
	}];
	
	
	[[signal retry] subscribeNext:^(id x) {
		NSLog(@"---try: %@",x);
	}];
	
}
```


### collect & aggregate

 collect 会将信号所有的值，整合为一个数组发送
 
 aggregate: 将所有的信号聚合为一个值发送 相当于scan 最后一次值


```objc
- (void)collect {
	
	[[@[@1, @2, @3, @4].rac_sequence.signal collect] subscribeNext:^(id x) {
		NSLog(@"%@ %@", [x class],x);//array
	}];
	
	
	[[@[@1, @2, @3, @4].rac_sequence.signal aggregateWithStart:@9 reduce:^id(NSNumber *running,NSNumber *next) {
		NSLog(@"%@----%@",running,next);
		return @(running.integerValue + next.integerValue);
	}] subscribeNext:^(id x) {
		NSLog(@"---> %@", x); // ----> 19
	}];
	
}
```


## 时间操作

### delay & throttle
 
 delay： 延时多少秒后，发送信号
 
 throttle： 节流，比如节流1秒，1秒内发送的信号，会取最新的信号


```objc
- (void)time {
	NSLog(@"---");
	[[@[@1, @2, @3, @4].rac_sequence.signal delay:1] subscribeNext:^(id x) {
		NSLog(@"---%@",x);
	}];
	
	RACSubject * subject = [RACSubject subject];
	
	[[subject throttle:1] subscribeNext:^(id x) {
		NSLog(@"%@", x);  // 注意结果是 2 4 不是1，3取连续发送的时候取最新的信号
	}];
	
	[subject sendNext:@1];
	[subject sendNext:@2];
	
	dispatch_after(dispatch_time(DISPATCH_TIME_NOW, (int64_t)(1 * NSEC_PER_SEC)), dispatch_get_main_queue(), ^{
		[subject sendNext:@3];
		[subject sendNext:@4];
	});
}
```


## 组合操作

### contact
 
 A 发送信号完成之后 B 才开始发送，按顺序contact
 
 例如 A 请求发送了， A请求的数据需要刷新UI 之后B请求拿到A请求的数据，发起请求
 之后再根据B请求的数据刷新界面
 
 注意两个点：
 1. 前一个信号，必须发送完成，下一个信号才会执行，顺序性
 2. 所有联结的信号订阅者都会收到信号

 
```objc
- (void)contact {
	
	RACSignal *signalA = [RACSignal createSignal:^RACDisposable *(id<RACSubscriber> subscriber) {
		
		[subscriber sendNext:@"A"];
		//必须发送complete B信号才会收到
		[subscriber sendCompleted];
		
		return nil;
	}];
	
	RACSignal *signalB = [RACSignal createSignal:^RACDisposable *(id<RACSubscriber> subscriber) {
		
		[subscriber sendNext:@"B"];
		[subscriber sendCompleted];
		
		return nil;
	}];
	
	//A， B 按顺序都会收到结果
	[[RACSignal concat:@[signalA, signalB]] subscribeNext:^(id x) {
		NSLog(@"%@", x);
	}];
	
}
```

### merge & zip
 merge : 将多个信号聚合为一个信号，没有先后顺序 (当做一个信号)
 
 zip: 将多个信号压缩为一个信号，所有的压缩的信号，都发送一次了，zip信号才会发送（一次为一组）


```objc
- (void)mergeAndZip {
	RACSubject *s1 = [RACSubject subject];
	RACSubject *s2 = [RACSubject subject];
	RACSubject *s3 = [RACSubject subject];
	
	[[RACSignal merge:@[s1, s2, s3]] subscribeNext:^(id x) {
		NSLog(@"%@", x);
	}];
	
	[[RACSignal zip:@[s1, s2, s3]] subscribeNext:^(id x) {
		NSLog(@"zip : %@", x);
		RACTupleUnpack(NSNumber *num1,NSNumber *num2,NSNumber *num3) = x;
		NSLog(@"zip[1]:%@",num2);
	}];
	
	
	
	[s1 sendNext:@1];
	[s2 sendNext:@2];
	[s3 sendNext:@3];
	
	
	[s1 sendNext:@6];
	[s2 sendNext:@7];
	[s3 sendNext:@8];
}
```

### then
 
 联结信号 A信号发送完成后，B信号才会发送
 
 结果为B的信号，如果A信号发送错误，直接结束
 
 可以理解为忽略前面信号，只取最后一个信号的contact


```objc
- (void)then {
	RACSignal *signalA = [RACSignal createSignal:^RACDisposable *(id<RACSubscriber> subscriber) {
		
		[subscriber sendNext:@"A"];
//		必须发送complete B信号才会收到
		[subscriber sendCompleted];
//		[subscriber sendError:nil]; 这里发送错误 订阅者会受到错误
		
		return nil;
	}];
	
	RACSignal *signalB = [RACSignal createSignal:^RACDisposable *(id<RACSubscriber> subscriber) {
		
		[subscriber sendNext:@"B"];
		[subscriber sendCompleted];
		
		return nil;
	}];
	
	RACSignal *signal = [signalA then:^RACSignal *{
		return signalB;
	}];
	
	[signal subscribeNext:^(id x) {
		NSLog(@"%@", x);
	} error:^(NSError *error) {
		NSLog(@"error");
	}];

}

```

### takeUntil & takeUntilReplacement
 
 takeUntil ： 信号B，发送信号之后，A将不再发送信号
 
 takeUntilReplacement： 信号B，发送信号后，订阅者将不再接收A的信号，代替为接收B信号


```objc
- (void)takeUntil {
	RACSubject *subject = [RACSubject subject];
	RACSubject *until = [RACSubject subject];
	
	[[subject takeUntil:until] subscribeNext:^(id x) {
		NSLog(@"%@", x);// 1 2 until 发送，信号会发送完成
	}];
	
	[[subject takeUntilReplacement:until] subscribeNext:^(id x) {
		NSLog(@"--%@", x);// 1 2 0 9 until信号发送后会取代原信号
	}];
	
	
	[subject sendNext:@1];
	[subject sendNext:@2];
	[until sendNext:@0];
	[subject sendNext:@3];
	[until sendNext:@9];
	[until sendCompleted];
	[subject sendNext:@4];
}

```



### combineLast
 
 聚合最新的信号值，聚合信号至少发送一次信号,常配合 reduce

 
 | letter |-  A  -  B  -  -   -   C  -    -   D
 | number |-  -  -   1  -  2   -   3   -   -    4
 | new   |-  -  -  -  B1 -  B2 C2 C3 -  D3 D4
 -------------- time ----------------------->
 
 
```objc
- (void)combineLast {
	
	[[RACSignal combineLatest:@[self.userNameTextField.rac_textSignal, self.passwordTextField.rac_textSignal]] subscribeNext:^(id x) {
		NSLog(@"%@----%@", x[0], x[1]);
	}];
	
	
	RACSignal *signal = [RACSignal combineLatest:@[field.rac_textSignal, field1.rac_textSignal] reduce:^id _Nonnull(NSString *account , NSString *pwd){
		
		return @(account.length && pwd.length);
	}];
	
	RAC(btn,enabled) = signal;
}
```

## 降阶操作

### ifThenElse
 
 if 信号A的bool值为yes 发送then信号 否则 发送 else信号
 
```objc
- (void)ifThenElse {
	RACSignal *signalA = @[@1, @2, @3].rac_sequence.signal;
	RACSignal *signalB = @[@4, @5, @6].rac_sequence.signal;
	RACSubject *condition = [RACSubject subject];
	
	[[RACSignal if:condition then:signalA else:signalB] subscribeNext:^(id x) {
		NSLog(@"%@", x); // 1,2,3 ====> 4,5,6  ====> 4,5,6
	}];
	
	[condition sendNext:@YES];
	
	dispatch_after(dispatch_time(DISPATCH_TIME_NOW, (int64_t)(2 * NSEC_PER_SEC)), dispatch_get_main_queue(), ^{
		[condition sendNext:@NO];
		dispatch_after(dispatch_time(DISPATCH_TIME_NOW, (int64_t)(4 * NSEC_PER_SEC)), dispatch_get_main_queue(), ^{
			[condition sendNext:@NO];
		});
	});
}
```

### switchToLast
 信号的降阶， 取最新的信号
 
 降阶： 原来是信号中的信号，降阶后为信号


```objc
	RACSubject *signalOfSignal = [RACSubject subject];
	RACSubject *subject1 = [RACSubject subject];
	RACSubject *subject2 = [RACSubject subject];
	RACSubject *subject3 = [RACSubject subject];
	
	[signalOfSignal.switchToLatest subscribeNext:^(id x) {
		NSLog(@"%@",x); // 3  ====>  2
	}];
	
	[signalOfSignal sendNext:subject1];
	[signalOfSignal sendNext:subject3];

	[subject2 sendNext:@2];//
	[subject3 sendNext:@3];
	[subject1 sendNext:@1];
	
	dispatch_after(dispatch_time(DISPATCH_TIME_NOW, (int64_t)(2 * NSEC_PER_SEC)), dispatch_get_main_queue(), ^{
		[signalOfSignal sendNext:subject2];
		[subject2 sendNext:@2];
		[subject3 sendNext:@3];
		[subject1 sendNext:@1];
	});
```


### flatten
//TODO 还没到搞懂




