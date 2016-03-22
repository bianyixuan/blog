# NSString属性什么时候用copy，什么时候用strong？

>我们在声明一个NSString属性的时候，对于其内存相关特性，通常有两种选择（基于ARC环境）：Strong与copy。那这两者有什么区别？什么时候该用Strong，什么时候该用copy呢？可以参考git中的demo[stringDemo](https://github.com/bianyixuan/stringDemo.git),或者看下面的例子:

<font color=green>示例：</font>

我们定义一个类，并为其声明两个字符串属性，如下所示:

	@interface TestStringClass ()
	@property (nonatomic, strong) NSString *strongString;
	@property (nonatomic, copy) NSString *copyedString;
	@end
	
上面的代码声明了两个字符串属性，其中一个内存特性是strong，一个是copy。下面我们来看看他们之间的区别。

首先，我们用一个不可变字符串来为这两个属性赋值，
	
	- (void)test {
	    NSString *string = [NSString stringWithFormat:@"abc"];
	    self.strongString = string;
	    self.copyedString = string;
	    NSLog(@"origin string: %p, %p", string, &string);
	    NSLog(@"strong string: %p, %p", _strongString, &_strongString);
	    NSLog(@"copy string: %p, %p", _copyedString, &_copyedString);
	}
	
其输出结果是：

	origin string: 0x7fe441592e20, 0x7fff57519a48
	strong string: 0x7fe441592e20, 0x7fe44159e1f8
	copy string: 0x7fe441592e20, 0x7fe44159e200
	
我们可以看到，这种情况下，不管是strong还是copy属性的对象，其指向的地址都是同一个，即为string指向的地址。如果我们换做MRC环境，打印string的引用计数的话，会看到其引用计数值是3，即strong操作和copy操作都使原字符串对象的引用计数加了1.

接下来，我们把string由不可变改为可变对象，看看会是神马结果。即将下面的这一句

	NSString *string = [NSString stringWithFormat:@"abc"];
	
改为：

	NSMutableString *string = [NSMutableString stringWithFormat:@"abc"];
	
其输出结果是:

	origin string: 0x7ff5f2e33c90, 0x7fff59937a48
	strong string: 0x7ff5f2e33c90, 0x7ff5f2e2aec8
	copy string: 0x7ff5f2e2aee0, 0x7ff5f2e2aed0
	
可以发现，此时copy属性字符串已不再指向string字符串对象，而是深拷贝了string字符串，并让_copyedStrign对象指向这个字符串。MRC环境下，打印两者的引用计数，可以看到string对象的引用计数为2，而_copyedString对象的引用计数是1.

此时，我们如果去修改string字符串的话，可以看到：因为_strongStrign与string是指向同一对象，所以_strongString的值也会跟着改变（需要注意的是，此时_strongString的类型实际上是NSMutableString，而不是NSString）;而_CopyedString是指向另一个对象的，所以并不会改变。

结论:

由于NSMutableString是NSString的子类，所以一个NSString指针可以指向NSMutableString对象，让我们的strongString指针指向一个可变字符串是OK的。

而上面的例子可以看出，当源字符串是NSString时，由于字符串是不可变的，所以，不管是strong还是copy属性的对象，都是指向源对象，copy操作只是做了次浅拷贝。

当源字符串是NSMutableString时，strong属性只是增加了源字符串的引用计数，而copy属性则是对源字符串做了次深拷贝，产生一个新的对象，且copy属性对象指向这个新的对象。另外需要注意的是，这个copy属性对象的类型始终是NSString，而不是NSMutableString，因此其是不可变的。

这里还有一个性能问题，即在源字符串是NSMutableString，strong是单纯的增加对象的引用计数，而copy操作是执行了一次深拷贝，所以性能上会有所差异。而如果源字符串是NSString时，则没有这个问题。

所以，在声明NSString属性时，到底是选择strong还是copy，可以根据实际情况来定。不过，一般我们将对象声明为NSString时，都不希望它改变，所以大多数情况下，我们建议用copy，以免因可变字符串的修改导致的一些非预期问题。

关于字符串的内存管理，还有些有意思的东西，可以参考[NSString特性分析学习](http://blog.cnbluebox.com/blog/2014/04/16/nsstringte-xing-fen-xi-xue-xi/)。

参考

1.	[NSString copy not copying?](http://stackoverflow.com/questions/2521468/nsstring-copy-not-copying)
2. 	[NSString特性分析学习](http://blog.cnbluebox.com/blog/2014/04/16/nsstringte-xing-fen-xi-xue-xi/)
3. 	[stringDemo](https://github.com/bianyixuan/stringDemo.git)