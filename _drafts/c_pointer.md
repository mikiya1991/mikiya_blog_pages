### 函数指针
C中函数是一个代码地址， 因此函数指针是指向该地址的一个指针。他们是不同的。

	typedef void (* pfunc)(void); 	//定义一个函数指针类型
	typedef void func(void);		//定义一个函数类型

	pfunc p1;
	func *p2;
	//两者意义相同

	//同时指针可以强制类型转换
	pfunc p3 = (pfunc)-1;
	pfunc p4 = (void (*)(void))-1;
	//两者相同

函数名也是一个函数指针，只不过它是一个**指针常量** 。因为函数就是一个地址，所以函数就是一个函数指针。
可以认为pfunc 和 func等同（调用和赋值中），但是申明时不等同。

	void Func(){
		return;
	}
	p1 = &Func;
	p1 = Func;
	//1合理，2是语法糖

	(*p1)();
	p1();
	//两者都可调用函数，1合理，2是语法糖。

详情参见文章[深入理解C语言函数指针](http://www.cnblogs.com/windlaughing/archive/2013/04/10/3012012.html)。


### 指针的赋值转换

## vim 插件的使用方式

### surrounding

ysiw(
ysiw)

cs([
ds[
yss(
vlllllS[

### easy motion
,,w ,,b

### ctrl p
,fu

### find

### vim.rc.local

### ag ack -i


### s/