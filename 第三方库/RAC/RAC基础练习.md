# 基础练习部分代码1

先练习代码，之后总结：
**注意：练习时没有加上循环引用处理**

```objc
	RACSignal *singal = self.userNameTextField.rac_textSignal;
	
	//map：转换（映射成return的类型）
	singal = [singal map:^id(NSString *value) {
		return @(value.length);
	}];
	
	
	//filter：过滤 (return YES 的值才会传递)
	RACSignal *enableSingal = [singal filter:^BOOL(NSNumber *value) {
		return value.integerValue >= 3;
	}];
	
	
	//skip： 跳过一个信号
	[[enableSingal skip:1] subscribeNext:^(id x) {
		NSLog(@"rac:%@",x);
	}];
	
	RACSignal *colorSiganl  = [enableSingal map:^(NSNumber *value) {
		return value.integerValue > 8 ? [UIColor greenColor] : [UIColor yellowColor];
	}];
	
	//RAC宏直接把信号输出的值，设置到对象的属性上
	RAC(self.userNameTextField,backgroundColor) = colorSiganl;
	
	RACSignal *passwordSignal = [[self.passwordTextField.rac_textSignal filter:^BOOL(NSString *value) {
		return value.length > 3;
	}] map:^id(NSString *value) {
		return @(value.length);
	}];
	
	//combineLatest(聚合信号) reduce（每当聚合的信号发送时，都会取最新的值，调用reduce） -> 聚合信号的值
	RAC(self.loginButton,backgroundColor) = [[RACSignal combineLatest:@[enableSingal, passwordSignal] reduce:^id(NSNumber *u, NSNumber *p){
		NSLog(@"来信号啦");
		return @(u.integerValue == 5 && p.integerValue == 6);
	}] map:^id(NSNumber *value) {
		return value.boolValue ? [UIColor greenColor] : [UIColor grayColor];
	}];
	
	
	
	RACSignal *loginsignal = [RACSignal createSignal:^RACDisposable *(id<RACSubscriber> subscriber) {
		
		dispatch_async(dispatch_get_global_queue(0, 0), ^{
			sleep(2);
			[subscriber sendNext:@(11)];
		});
		return nil;
	}];
	
	
	//在信号 sendNext之前 先做 block里的事
	[[[[[self.loginButton  rac_signalForControlEvents:UIControlEventTouchUpInside] doNext:^(id x) {
		self.view.backgroundColor = [UIColor redColor];
		self.loginButton.userInteractionEnabled = NO;
	}]
	//flattenMap 将信号 映射成 其他信号
    flattenMap:^RACStream *(id value) {
		return loginsignal;
	}]
	  // deliverOn : 切换线程
    deliverOn:[RACScheduler mainThreadScheduler]]
	subscribeNext:^(NSNumber *x) {
		if(x.integerValue == 11){
			NSLog(@"登录成功");
			self.view.backgroundColor = [UIColor whiteColor];
			self.loginButton.userInteractionEnabled = YES;
		}
	}
	 completed:^{
		//测试发现这里不会执行，实际上是flattenMap之前信号的订阅
	}];
	
```


练习二：


```objc
//RACCommand : 可以手动触发信号
	RACCommand *commond = [[RACCommand  alloc] initWithSignalBlock:^RACSignal *(NSNumber *input) {
		RACSignal *sinal = [RACSignal createSignal:^RACDisposable *(id<RACSubscriber> subscriber) {
			
			if(input.integerValue != 7) return nil;
			
			//UIAlertView: rac_buttonClickedSignal
			UIAlertView *alertView = [[UIAlertView alloc] initWithTitle:@"确定吗" message:@"确定吗" delegate:nil cancelButtonTitle:@"NO" otherButtonTitles:@"YES", nil];
			
			[[alertView rac_buttonClickedSignal] subscribeNext:^(NSNumber *x) {
				if(x.integerValue == 1) {
					[subscriber sendNext:@(YES)];
					[subscriber sendCompleted];
				}else {
					[subscriber sendError:nil];
				}
			}];
			[alertView show];
			return nil;
		}];
		
		return sinal;
	}];
	
	@weakify(self)
	dispatch_after(dispatch_time(DISPATCH_TIME_NOW, (int64_t)(2 * NSEC_PER_SEC)), dispatch_get_main_queue(), ^{
		//RACCommand execute 执行信号block
		[[[[[[commond execute:@(7)]
		//上个信号sendCompleted之后，会连接成 then 里的信号
	     then:^RACSignal *{
			return self.passwordTextField.rac_textSignal;}]
		filter:^BOOL(NSString *value) {
			return value.length > 0;
		}]
		//节流1秒钟，1秒钟内有信号，1秒钟过后只取最后一个信号，比如防止搜索框疯狂修改值
		 throttle:1 ]
		flattenMap:^RACStream *(NSString *value) {
			
			
			RACSignal *serchSiganl = [RACSignal createSignal:^RACDisposable *(id<RACSubscriber> subscriber) {
				
				[subscriber sendNext:@{@"1":@"A",@"2":@"B"}[value]];
				
				return nil;
			}];
			
			return serchSiganl;
		}] subscribeNext:^(NSString *x) {
			@strongify(self)
			[self.loginButton setTitle:x forState:UIControlStateNormal];
		}] ;
	});
```

## raccommand


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


