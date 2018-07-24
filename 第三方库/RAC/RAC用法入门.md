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


