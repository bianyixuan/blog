    - (void)performSelector:(SEL)aSelector withObject:(id)anArgument afterDelay:(NSTimeInterval)delay; 

#关于performSelectorXXX的延迟使用

这个方法是单线程的，也就是说只有当前调用此方法的函数执行完毕后，selector方法才会被调用。

比如：


    - (void)changeStr:(NSString *)string
    {
        NSLog(@"changeText:%@",(NSString *)string);
    }

    - (void)changePopoverSize
    {   
        [self performSelector:@selector(changeStr:) withObject:@"Happy Brandon" afterDelay:1];

        NSLog(@"changePopoverSize-----start");
        sleep(5);
        NSLog(@"changePopoverSize-----end");
    }

执行结果（注意时间）：

        2016-03-14 21:55:22.305 Demo[31200:6870787] changePopoverSize-----start
        2016-03-14 21:55:27.307 Demo[31200:6870787] changePopoverSize-----end
        2016-03-14 21:55:27.317 Demo[31200:6870787] changeText:Happy Brandon
        

在多线程方法中不会执行此方法，比如：

    -(void) testDispatch_Barrier{
        dispatch_queue_t gcd = dispatch_queue_create("这是并发队列", DISPATCH_QUEUE_CONCURRENT);
        dispatch_async(gcd, ^{
        NSLog(@"b0");
        //这个selector不会执行
         [self performSelector:@selector(changeStr:) withObject:@"此方法执行不了" afterDelay:3];
        NSLog(@"changePopoverSize-----start");
        NSLog(@"changePopoverSize-----end");
        });
    }


如果要想多线程的话，可以是使用

        - (void)performSelectorInBackground:(SEL)aSelector withObject:(id)arg
或者

        - (void)performSelectorOnMainThread:(SEL)aSelector withObject:(id)arg waitUntilDone:(BOOL)wait;
        
代码如下：        

    -(void) testDispatch_Barrier{
        dispatch_queue_t gcd = dispatch_queue_create("这是并发队列", DISPATCH_QUEUE_CONCURRENT);
        dispatch_async(gcd, ^{
        NSLog(@"b0");
        [self performSelectorOnMainThread:@selector(changeStr:) withObject:@"onMainThread正常执行" waitUntilDone:YES];
       // [self performSelectorInBackground:@selector(changeStr:) withObject:@"inBackground正常执行"];
        NSLog(@"changePopoverSize-----start");
        NSLog(@"changePopoverSize-----end");
        });
    }
  
  执行结果如下：
  
    2016-03-14 22:12:36.761 Demo[31355:6879429] changeText:onMainThread正常执行
    2016-03-14 22:12:36.762 Demo[31355:6879570] changePopoverSize-----start
    2016-03-14 22:12:36.764 Demo[31355:6879570] changePopoverSize-----end
    
可以看出，当waitUnitDone:YES的时候，等changeStr方法执行完成之后才会继续后面的代码执行。当waitUnitDone:NO的时候，执行结果如下：
   
    2016-03-14 22:08:59.377 Demo[31297:6877086] b0
    2016-03-14 22:08:59.378 Demo[31297:6877086] changePopoverSize-----start
    2016-03-14 22:08:59.379 Demo[31297:6877086] changePopoverSize-----end
    2016-03-14 22:08:59.386 Demo[31297:6876909] changeText:onMainThread正常执行